#include <open62541/client_config_default.h>
#include <open62541/client_highlevel.h>
#include <string.h>
#include <stdint.h>

// Function to connect to OPC UA server
// Returns: 0 on success, error code otherwise
uint32_t OPCUA_Connect(const char* serverUrl, uint32_t timeoutMs, UA_Client** client) {
    if (!serverUrl || !client) return 1; // Invalid parameters

    // Initialize client
    *client = UA_Client_new();
    if (!*client) return 2; // Memory allocation failed

    UA_ClientConfig_setDefault(UA_Client_getConfig(*client));

    // Set timeout
    UA_Client_getConfig(*client)->timeout = timeoutMs;

    // Connect to server
    UA_StatusCode status = UA_Client_connect(*client, serverUrl);
    if (status != UA_STATUSCODE_GOOD) {
        UA_Client_delete(*client);
        *client = NULL;
        return status; // Return OPC UA status code
    }

    return 0; // Success
}

// Function to disconnect from OPC UA server
void OPCUA_Disconnect(UA_Client* client) {
    if (client) {
        UA_Client_disconnect(client);
        UA_Client_delete(client);
    }
}
