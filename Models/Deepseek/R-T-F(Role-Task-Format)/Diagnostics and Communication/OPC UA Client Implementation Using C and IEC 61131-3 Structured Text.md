#include <open62541/client.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct {
    UA_Client *client;
    char serverUrl[256];
    int done;
    int busy;
    int error;
    unsigned long errorId;
} OpcUaClient;

OpcUaClient* opcua_client_create(const char* serverUrl) {
    OpcUaClient* clientData = malloc(sizeof(OpcUaClient));
    if (!clientData) return NULL;
    clientData->client = UA_Client_new();
    strncpy(clientData->serverUrl, serverUrl, sizeof(clientData->serverUrl) - 1);
    clientData->serverUrl[sizeof(clientData->serverUrl) - 1] = '\0';
    clientData->done = 0;
    clientData->busy = 0;
    clientData->error = 0;
    clientData->errorId = 0;
    return clientData;
}

void opcua_client_destroy(OpcUaClient* clientData) {
    if (clientData) {
        UA_Client_delete(clientData->client);
        free(clientData);
    }
}

int opcua_client_connect(OpcUaClient* clientData) {
    if (!clientData || !clientData->client) return -1;
    clientData->busy = 1;
    clientData->done = 0;
    clientData->error = 0;
    clientData->errorId = 0;

    UA_StatusCode retval = UA_Client_connect(clientData->client, clientData->serverUrl);
    if (retval != UA_STATUSCODE_GOOD) {
        clientData->busy = 0;
        clientData->error = 1;
        clientData->errorId = retval;
        return -1;
    }

    clientData->busy = 0;
    clientData->done = 1;
    return 0;
}

int opcua_client_is_connected(OpcUaClient* clientData) {
    if (!clientData || !clientData->client) return 0;
    return UA_Client_getState(clientData->client) == UA_CLIENTSTATE_CONNECTED;
}
