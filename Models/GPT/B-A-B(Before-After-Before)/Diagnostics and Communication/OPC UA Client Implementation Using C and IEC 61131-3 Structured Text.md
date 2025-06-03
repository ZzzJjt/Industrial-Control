// File: opcua_client_connect.c
#include <open62541/client.h>
#include <open62541/client_config_default.h>
#include <string.h>

#define OPCUA_SUCCESS 0
#define OPCUA_TIMEOUT 1
#define OPCUA_CONNECT_FAIL 2

// Simple connect-only logic, no read/write for brevity
unsigned int opcua_connect(const char* serverUrl, unsigned int timeoutMs) {
    UA_Client *client = UA_Client_new();
    UA_ClientConfig_setDefault(UA_Client_getConfig(client));

    UA_StatusCode status = UA_Client_connect(client, serverUrl);
    if (status != UA_STATUSCODE_GOOD) {
        UA_Client_delete(client);
        return OPCUA_CONNECT_FAIL;
    }

    // Wait loop to simulate timeout-based monitoring
    unsigned int waited = 0;
    while (waited < timeoutMs) {
        UA_StatusCode s = UA_Client_run_iterate(client, false);
        if (s != UA_STATUSCODE_GOOD) {
            UA_Client_disconnect(client);
            UA_Client_delete(client);
            return OPCUA_CONNECT_FAIL;
        }
        UA_sleep_ms(50);
        waited += 50;
    }

    UA_Client_disconnect(client);
    UA_Client_delete(client);
    return OPCUA_SUCCESS;
}


FUNCTION_BLOCK OPCUA_Connect_FB
VAR_INPUT
    Execute     : BOOL;
    ServerUrl   : STRING(255);
    Timeout     : TIME;
END_VAR
VAR_OUTPUT
    Done        : BOOL;
    Busy        : BOOL;
    Error       : BOOL;
    ErrorID     : DWORD;
END_VAR
VAR
    InternalBusy   : BOOL := FALSE;
    StartTime      : TIME;
    TimeoutMs      : DWORD;
    RisingEdge     : BOOL := FALSE;
END_VAR

// External declaration
{extern}
FUNCTION opcua_connect_c : DWORD
VAR_INPUT
    serverurl : STRING(255);
    timeoutMs : DWORD;
END_VAR
