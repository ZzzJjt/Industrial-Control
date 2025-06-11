#include <open62541/client.h>
#include <open62541/client_config_default.h>
#include <open62541/client_highlevel.h>
#include <string.h>

typedef struct {
    UA_Client *client;
    bool connected;
} OpcUaClientContext;

static OpcUaClientContext ctx = {0};

// Initialize and start connection
int OpcUa_StartConnection(const char* serverUrl, UA_UInt32 timeout) {
    if (ctx.client != NULL)
        return -1; // Already in use

    ctx.client = UA_Client_new();
    UA_ClientConfig_setDefault(UA_Client_getConfig(ctx.client));
    UA_Client_getConfig(ctx.client)->timeout = timeout;

    UA_StatusCode retval = UA_Client_connect(ctx.client, serverUrl);
    if (retval != UA_STATUSCODE_GOOD) {
        UA_Client_delete(ctx.client);
        ctx.client = NULL;
        return retval;
    }

    ctx.connected = true;
    return UA_STATUSCODE_GOOD;
}

// Check connection status
bool OpcUa_IsConnected(void) {
    if (!ctx.client)
        return false;
    return ctx.connected;
}

// Disconnect and clean up
void OpcUa_Disconnect(void) {
    if (ctx.client) {
        UA_Client_disconnect(ctx.client);
        UA_Client_delete(ctx.client);
        ctx.client = NULL;
    }
    ctx.connected = false;
}

#ifndef OPCUA_INTERFACE_H
#define OPCUA_INTERFACE_H

#ifdef __cplusplus
extern "C" {
#endif

int OpcUa_StartConnection(const char* serverUrl, unsigned int timeout);
int OpcUa_IsConnected(void);
void OpcUa_Disconnect(void);

#ifdef __cplusplus
}
#endif

#endif // OPCUA_INTERFACE_H

FUNCTION_BLOCK FB_OpcUaClient
VAR_INPUT
    Execute: BOOL := FALSE;           // Rising edge starts connection
    ServerUrl: STRING[255] := '';     // OPC UA server URL
    Timeout: TIME := T#5000ms;        // Connection timeout
END_VAR

VAR_OUTPUT
    Done: BOOL := FALSE;              // Operation completed successfully
    Busy: BOOL := FALSE;              // Operation in progress
    Error: BOOL := FALSE;             // An error occurred
    ErrorID: DWORD := 0;              // Error code from OPC UA
END_VAR

VAR
    lastExecute: BOOL := FALSE;
    startTime: TIME := T#0ms;
    isConnected: BOOL := FALSE;
END_VAR

// Reset outputs
Done := FALSE;
Error := FALSE;

CASE TRUE OF
    NOT Execute:
        // Idle state
        Busy := FALSE;
        IF lastExecute THEN
            // If execution just ended, disconnect
            OpcUa_Disconnect();
            lastExecute := FALSE;
        END_IF;

    Execute AND NOT Busy:
        // Start operation on rising edge
        IF NOT lastExecute THEN
            startTime := TIME();
            Busy := TRUE;
            Done := FALSE;

            // Convert timeout from TIME to ms (approximate)
            timeoutMs := UDINT_TO_UINT((Timeout / TIME#1MS));

            // Call external C function
            errorCode := OpcUa_StartConnection(ADR(ServerUrl), timeoutMs);
            IF errorCode <> 0 THEN
                Error := TRUE;
                ErrorID := DWORD(errorCode);
                Busy := FALSE;
                Done := FALSE;
            ELSE
                isConnected := TRUE;
            END_IF;
        END_IF;
        lastExecute := TRUE;

    Busy:
        // Check if still busy
        isConnected := OpcUa_IsConnected();
        IF isConnected THEN
            Busy := FALSE;
            Done := TRUE;
        ELSIF (TIME() - startTime) > Timeout THEN
            // Timeout expired
            Busy := FALSE;
            Error := TRUE;
            ErrorID := 16#8001; // Custom timeout error code
            OpcUa_Disconnect();
        END_IF;
END_CASE;
