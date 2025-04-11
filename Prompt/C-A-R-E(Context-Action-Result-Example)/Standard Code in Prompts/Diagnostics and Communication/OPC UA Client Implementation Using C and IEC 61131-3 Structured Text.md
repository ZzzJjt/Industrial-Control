```
#include <open62541/client_high_level_common.h>
#include <open62541/client_config_default.h>
#include <open62541/client_subscriptions.h>
#include <open62541/client_async_operations.h>
#include <string.h>

#define SERVER_URL_MAX_LENGTH 255

UA_StatusCode connectToServer(const char *serverUrl, UA_Client *client) {
    UA_ClientConfig_setDefault(UA_Client_getConfig(client));
    UA_Client_setEndpointUrl(client, serverUrl);
    return UA_Client_connect(client);
}

void disconnectFromServer(UA_Client *client) {
    UA_Client_disconnect(client);
}

int main() {
    const char *serverUrl = "opc.tcp://localhost:4840/freeopcua/server/";
    UA_Client client;
    UA_Client_init(&client);

    if (connectToServer(serverUrl, &client) == UA_STATUSCODE_GOOD) {
        // Perform operations here...
        // For example, read a variable or browse nodes
        // UA_Client_readValue(...)

        // Disconnect when done
        disconnectFromServer(&client);
    } else {
        printf("Failed to connect to server.\n");
    }

    UA_Client_delete(&client);
    return 0;
}
```
```
FUNCTION_BLOCK OPC_UA_CLIENT
VAR_INPUT
    Execute : BOOL;
    ServerUrl : STRING(255);
    Timeout : TIME;
VAR_OUTPUT
    Done : BOOL;
    Busy : BOOL;
    Error : BOOL;
    ErrorID : DWORD;
VAR
    CClient : ANY; // Placeholder for the C client object
    OperationResult : UINT;
END_VAR

// Initialize client only once
IF NOT CClient.INITIALIZED THEN
    CClient.INIT(ServerUrl);
    CClient.INITIALIZED := TRUE;
END_IF

IF Execute THEN
    // Execute the operation (e.g., read variable)
    OperationResult := CClient.EXECUTE(OperationType, VariablePath);
    
    IF OperationResult = UA_STATUSCODE_GOOD THEN
        Done := TRUE;
        Busy := FALSE;
        Error := FALSE;
    ELSE
        Done := FALSE;
        Busy := FALSE;
        Error := TRUE;
        ErrorID := OperationResult;
    END_IF
ELSE
    Done := FALSE;
    Busy := TRUE;
    Error := FALSE;
END_IF

// Example pseudo-functions for demonstration
PROCEDURE CClient_INIT(ServerURL : STRING(255))
    // Initializes the C client with the given server URL
    // This is a placeholder for the actual initialization logic
END_PROC

PROCEDURE CClient_EXECUTE(OperationType : UINT; VariablePath : STRING(255)) : UINT
    // Executes the specified operation on the OPC UA server
    // Returns the operation result
    RETURN UA_STATUSCODE_GOOD;
END_PROC
END_FUNCTION_BLOCK
```
