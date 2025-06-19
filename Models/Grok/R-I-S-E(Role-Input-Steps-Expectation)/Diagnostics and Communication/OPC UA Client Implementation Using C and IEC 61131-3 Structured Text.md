#include <open62541/client_config_default.h>
#include <open62541/client_highlevel.h>
#include <string.h>
#include <stdint.h>

// Client context
static UA_Client* client = NULL;
static UA_StatusCode last_status = UA_STATUSCODE_GOOD;

// Initialize client and connect to server
int32_t opcua_connect(const char* server_url, int32_t timeout_ms) {
    if (client != NULL) {
        return UA_STATUSCODE_BADSESSIONACTIVE; // Client already initialized
    }

    client = UA_Client_new();
    if (!client) {
        return UA_STATUSCODE_BADOUTOFMEMORY;
    }

    UA_ClientConfig_setDefault(UA_Client_getConfig(client));
    UA_ClientConfig* config = UA_Client_getConfig(client);
    config->timeout = timeout_ms;

    last_status = UA_Client_connect(client, server_url);
    if (last_status != UA_STATUSCODE_GOOD) {
        UA_Client_delete(client);
        client = NULL;
        return last_status;
    }

    return UA_STATUSCODE_GOOD;
}

// Disconnect and clean up
int32_t opcua_disconnect(void) {
    if (client == NULL) {
        return UA_STATUSCODE_BADSESSIONCLOSED;
    }

    last_status = UA_Client_disconnect(client);
    UA_Client_delete(client);
    client = NULL;
    return last_status;
}

// Get connection status
int32_t opcua_get_status(void) {
    if (client == NULL) {
        return UA_STATUSCODE_BADSESSIONCLOSED;
    }
    return last_status;
}
