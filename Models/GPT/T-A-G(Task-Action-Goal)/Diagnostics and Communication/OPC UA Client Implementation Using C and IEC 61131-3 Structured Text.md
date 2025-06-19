(* C-Layer OPC UA Client Stub using open62541 *)
// This would be compiled separately and linked into the PLC runtime

#include <open62541/client_config_default.h>
#include <open62541/client_highlevel.h>

UA_Client *client;
int opcua_connect(const char *url, unsigned int timeout_ms) {
    client = UA_Client_new();
    UA_ClientConfig_setDefault(UA_Client_getConfig(client));
    UA_StatusCode status = UA_Client_connect_timeout(client, url, NULL, timeout_ms);
    return (int)status;
}

void opcua_disconnect() {
    UA_Client_disconnect(client);
    UA_Client_delete(client);
}


(* IEC 61131-3 Structured Text Wrapper *)
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
    prevExecute  : BOOL := FALSE;
    state        : INT := 0;
    timerStart   : TIME;
    elapsedTime  : TIME;
    returnCode   : INT;
END_VAR

(* Rising edge detection *)
IF Execute AND NOT prevExecute THEN
    state := 1;
    Busy := TRUE;
    Done := FALSE;
    Error := FALSE;
    ErrorID := 0;
END_IF
prevExecute := Execute;

(* OPC UA client state machine *)
CASE state OF

    1: // Connect
        returnCode := opcua_connect(ADR(ServerUrl), TO_UINT(Timeout));
        IF returnCode = 0 THEN
            state := 2;
        ELSE
            Error := TRUE;
            ErrorID := DWORD(returnCode);
            Busy := FALSE;
            state := 0;
        END_IF

    2: // Connected
        Done := TRUE;
        Busy := FALSE;
        state := 0;

END_CASE
