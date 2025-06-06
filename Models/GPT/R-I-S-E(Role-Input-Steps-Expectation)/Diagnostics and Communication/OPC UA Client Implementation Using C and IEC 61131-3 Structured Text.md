#include <open62541/client.h>
#include <open62541/client_config_default.h>
#include <string.h>
#include <time.h>

typedef struct {
    UA_Client *client;
    UA_Boolean connected;
    UA_StatusCode status;
} OPCUA_ClientHandle;

OPCUA_ClientHandle g_clientHandle = {0};

int opcua_connect(const char *endpoint, unsigned int timeout_ms) {
    if(g_clientHandle.connected)
        return UA_STATUSCODE_GOOD; // Already connected

    g_clientHandle.client = UA_Client_new();
    UA_ClientConfig_setDefault(UA_Client_getConfig(g_clientHandle.client));

    // Set timeout
    g_clientHandle.client->config.timeout = timeout_ms;

    g_clientHandle.status = UA_Client_connect(g_clientHandle.client, endpoint);
    g_clientHandle.connected = (g_clientHandle.status == UA_STATUSCODE_GOOD);
    return g_clientHandle.status;
}

int opcua_disconnect() {
    if(g_clientHandle.connected) {
        UA_Client_disconnect(g_clientHandle.client);
        UA_Client_delete(g_clientHandle.client);
        g_clientHandle.connected = false;
        return UA_STATUSCODE_GOOD;
    }
    return UA_STATUSCODE_BADCONNECTIONCLOSED;
}

int opcua_get_status() {
    return g_clientHandle.status;
}

FUNCTION_BLOCK FB_OPCUA_Client
VAR_INPUT
    Execute    : BOOL;
    ServerUrl  : STRING(255);
    Timeout    : TIME;
END_VAR
VAR_OUTPUT
    Done       : BOOL;
    Busy       : BOOL;
    Error      : BOOL;
    ErrorID    : DWORD;
END_VAR
VAR
    EdgeDetect : R_TRIG;
    tStart     : TIME;
    tNow       : TIME;
    State      : INT;
END_VAR

// External C Function Declarations
FUNCTION opcua_connect: DINT
    VAR_INPUT
        endpoint : POINTER TO BYTE;
        timeout  : UDINT;
    END_VAR;

FUNCTION opcua_disconnect: DINT;
FUNCTION opcua_get_status: DINT;

METHOD Main
EdgeDetect(CLK := Execute);

CASE State OF
0: // Idle
    Done := FALSE;
    Busy := FALSE;
    Error := FALSE;
    ErrorID := 0;

    IF EdgeDetect.Q THEN
        tStart := TIME(); // System time capture
        Busy := TRUE;
        State := 1;
    END_IF

1: // Connecting
    tNow := TIME();
    IF (tNow - tStart) > Timeout THEN
        Busy := FALSE;
        Error := TRUE;
        ErrorID := 16#DEAD0001; // Timeout error
        State := 0;
    ELSE
        ErrorID := opcua_connect(ADR(ServerUrl), TO_UDINT(TO_DINT(Timeout) / 1000));
        IF ErrorID = 0 THEN
            Done := TRUE;
            Busy := FALSE;
            Error := FALSE;
            State := 0;
        ELSIF ErrorID <> 0 THEN
            Busy := FALSE;
            Error := TRUE;
            State := 0;
        END_IF
    END_IF
END_CASE
