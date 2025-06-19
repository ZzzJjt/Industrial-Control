#include <open62541/client_config_default.h>
#include <open62541/client_highlevel.h>
#include <string.h>
#include <stdio.h>

typedef enum {
    OPCUA_STATUS_IDLE = 0,
    OPCUA_STATUS_BUSY,
    OPCUA_STATUS_DONE,
    OPCUA_STATUS_ERROR
} OPCUA_STATUS;

typedef struct {
    char serverUrl[256];
    unsigned int timeout_ms;
    OPCUA_STATUS status;
    unsigned int errorCode;
} OPCUA_Client;

OPCUA_Client client;

int opcua_connect(OPCUA_Client *clientCtx) {
    UA_Client *uaClient = UA_Client_new();
    UA_ClientConfig_setDefault(UA_Client_getConfig(uaClient));

    UA_StatusCode retval = UA_Client_connect(uaClient, clientCtx->serverUrl);
    if(retval != UA_STATUSCODE_GOOD) {
        UA_Client_delete(uaClient);
        clientCtx->status = OPCUA_STATUS_ERROR;
        clientCtx->errorCode = retval;
        return -1;
    }

    // Optional: Read a variable or perform operations here

    UA_Client_disconnect(uaClient);
    UA_Client_delete(uaClient);

    clientCtx->status = OPCUA_STATUS_DONE;
    clientCtx->errorCode = 0;
    return 0;
}

// PLC calls this function with Execute == 1
int opcua_execute(const char *serverUrl, unsigned int timeout_ms, unsigned int *errorCode) {
    strncpy(client.serverUrl, serverUrl, 255);
    client.timeout_ms = timeout_ms;
    client.status = OPCUA_STATUS_BUSY;

    int result = opcua_connect(&client);
    *errorCode = client.errorCode;

    return client.status;
}


FUNCTION_BLOCK FB_OpcUaClient
VAR_INPUT
    Execute   : BOOL;
    ServerUrl : STRING(255);
    Timeout   : TIME;
END_VAR
VAR_OUTPUT
    Done     : BOOL;
    Busy     : BOOL;
    Error    : BOOL;
    ErrorID  : DWORD;
END_VAR
VAR
    status      : INT;
    errorCode   : DWORD;
    prevExecute : BOOL;
END_VAR

// External call (bind to opcua_execute in opcua_client.c)
FUNCTION opcua_execute : INT
VAR_INPUT
    serverUrl : STRING;
    timeoutMs : UDINT;
    errorCode : POINTER TO UDINT;
END_VAR

IF Execute AND NOT prevExecute THEN
    Busy := TRUE;
    Done := FALSE;
    Error := FALSE;
    ErrorID := 0;

    status := opcua_execute(ServerUrl := ServerUrl, timeoutMs := TO_UDINT(Timeout), errorCode := ADR(errorCode));

    CASE status OF
        0:  // Idle – No operation
            Done := FALSE;
            Busy := FALSE;
            Error := FALSE;
        1:  // Busy
            Busy := TRUE;
        2:  // Done
            Done := TRUE;
            Busy := FALSE;
            Error := FALSE;
        3:  // Error
            Done := FALSE;
            Busy := FALSE;
            Error := TRUE;
            ErrorID := errorCode;
    END_CASE;
END_IF;

prevExecute := Execute;
