#include <open62541/client_config_default.h>
#include <open62541/client_highlevel.h>
#include <open62541/plugin/log_stdout.h>
#include <string.h>

typedef struct {
    UA_Client *client;
    UA_Boolean isConnected;
    UA_UInt32 errorCode;
} OPCUA_Client;

OPCUA_Client* opcua_client_init() {
    OPCUA_Client *ctx = (OPCUA_Client*)UA_calloc(1, sizeof(OPCUA_Client));
    if (!ctx) return NULL;
    
    ctx->client = UA_Client_new();
    if (!ctx->client) {
        UA_free(ctx);
        return NULL;
    }
    
    UA_ClientConfig_setDefault(UA_Client_getConfig(ctx->client));
    ctx->isConnected = UA_FALSE;
    ctx->errorCode = 0;
    return ctx;
}

UA_StatusCode opcua_client_connect(OPCUA_Client *ctx, const char *serverUrl, UA_UInt32 timeoutMs) {
    if (!ctx || !ctx->client || !serverUrl) {
        ctx->errorCode = 0x80010000; // Invalid parameters
        return UA_STATUSCODE_BADINVALIDARGUMENT;
    }
    
    // Disconnect if already connected
    if (ctx->isConnected) {
        UA_Client_disconnect(ctx->client);
        ctx->isConnected = UA_FALSE;
    }
    
    // Set timeout
    UA_ClientConfig *config = UA_Client_getConfig(ctx->client);
    config->timeout = timeoutMs;
    
    // Connect to server
    UA_StatusCode status = UA_Client_connect(ctx->client, serverUrl);
    if (status == UA_STATUSCODE_GOOD) {
        ctx->isConnected = UA_TRUE;
        ctx->errorCode = 0;
    } else {
        ctx->isConnected = UA_FALSE;
        ctx->errorCode = status;
    }
    
    return status;
}

void opcua_client_cleanup(OPCUA_Client *ctx) {
    if (!ctx) return;
    
    if (ctx->client) {
        UA_Client_disconnect(ctx->client);
        UA_Client_delete(ctx->client);
    }
    
    UA_free(ctx);
}

UA_UInt32 opcua_client_get_error(OPCUA_Client *ctx) {
    return ctx ? ctx->errorCode : 0x80020000; // Null context
}

UA_Boolean opcua_client_is_connected(OPCUA_Client *ctx) {
    return ctx ? ctx->isConnected : UA_FALSE;
}
