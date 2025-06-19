#include <open62541/client_config_default.h>
#include <open62541/client_highlevel.h>
#include <string.h>

/*
 * Function: opcua_connect
 * Purpose: Establishes an OPC UA client connection to a server
 * Parameters:
 *   server_url - Null-terminated string (e.g., "opc.tcp://localhost:4840")
 *   timeout_ms - Connection timeout in milliseconds
 * Returns:
 *   error_id - 0x0000 (success), 0x0001 (invalid URL), 0x0002 (connection failed),
 *              0x0003 (timeout), 0xFFFF (unknown error)
 */
unsigned long opcua_connect(const char* server_url, unsigned long timeout_ms) {
    // Validate inputs
    if (!server_url || strlen(server_url) == 0 || strncmp(server_url, "opc.tcp://", 10) != 0) {
        return 0x0001; // Invalid URL
    }

    // Create and configure client
    UA_Client* client = UA_Client_new();
    if (!client) {
        return 0xFFFF; // Unknown error
    }
    UA_ClientConfig_setDefault(UA_Client_getConfig(client));

    // Set timeout (open62541 uses milliseconds)
    UA_Client_getConfig(client)->timeout = timeout_ms;

    // Attempt connection
    UA_StatusCode status = UA_Client_connect(client, server_url);
    if (status == UA_STATUSCODE_GOOD) {
        // Connection successful
        UA_Client_delete(client);
        return 0x0000;
    } else if (status == UA_STATUSCODE_BADTIMEOUT) {
        UA_Client_delete(client);
        return 0x0003; // Timeout
    } else {
        UA_Client_delete(client);
        return 0x0002; // Connection failed
    }
}
