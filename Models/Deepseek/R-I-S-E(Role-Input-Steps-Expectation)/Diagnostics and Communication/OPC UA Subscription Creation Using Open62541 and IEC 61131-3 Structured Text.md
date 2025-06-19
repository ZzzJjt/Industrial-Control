#include <open62541/client.h>
#include <open62541/client_config_default.h>
#include <string.h>

typedef struct {
    UA_Client* client;           // Associated client
    UA_UInt32 subscriptionId;    // Subscription ID
    bool active;
} OpcUaSubscriptionContext;

// Static context pool for up to 10 simultaneous subscriptions
#define MAX_SUBSCRIPTIONS 10
static OpcUaSubscriptionContext subCtxPool[MAX_SUBSCRIPTIONS] = {0};

// Helper: Get context by handle
OpcUaSubscriptionContext* getSubscriptionContext(UA_UInt32 hdl) {
    if (hdl >= MAX_SUBSCRIPTIONS)
        return NULL;
    return &subCtxPool[hdl];
}

// Create a new subscription
UA_UInt32 OpcUa_CreateSubscription(
    UA_Client* client,
    UA_Byte priority,
    UA_Double publishingInterval,
    UA_StatusCode* outError
) {
    for (UA_UInt32 i = 0; i < MAX_SUBSCRIPTIONS; i++) {
        OpcUaSubscriptionContext* ctx = &subCtxPool[i];
        if (!ctx->active) {
            UA_CreateSubscriptionRequest request = UA_CreateSubscriptionRequest_default();
            request.priority = priority;
            request.publishingInterval = publishingInterval;

            UA_CreateSubscriptionResponse response = UA_Client_Service_Subscription_createSubscription(client, UA_Client_getConfig(client)->clientContext, request);

            if (response.responseHeader.serviceResult != UA_STATUSCODE_GOOD) {
                *outError = response.responseHeader.serviceResult;
                return 0;
            }

            ctx->client = client;
            ctx->subscriptionId = response.subscriptionId;
            ctx->active = true;

            *outError = UA_STATUSCODE_GOOD;
            return i; // Return handle
        }
    }

    *outError = UA_STATUSCODE_BADINTERNALERROR; // No free slot
    return 0;
}

// Update subscription publishing interval
UA_StatusCode OpcUa_UpdateSubscriptionInterval(UA_UInt32 hdl, UA_Double newInterval) {
    OpcUaSubscriptionContext* ctx = getSubscriptionContext(hdl);
    if (!ctx || !ctx->active)
        return UA_STATUSCODE_BADSUBSCRIPTIONIDINVALID;

    UA_ModifySubscriptionRequest request;
    UA_ModifySubscriptionRequest_init(&request);
    request.subscriptionId = ctx->subscriptionId;
    request.publishingInterval = newInterval;

    UA_ModifySubscriptionResponse response = UA_Client_Service_Subscription_modifySubscription(ctx->client, ctx->client->config.clientContext, request);

    return response.responseHeader.serviceResult;
}

// Delete subscription
void OpcUa_DeleteSubscription(UA_UInt32 hdl) {
    OpcUaSubscriptionContext* ctx = getSubscriptionContext(hdl);
    if (!ctx || !ctx->active)
        return;

    UA_DeleteSubscriptionsRequest request;
    UA_DeleteSubscriptionsRequest_init(&request);
    request.subscriptionIdsSize = 1;
    request.subscriptionIds = &ctx->subscriptionId;

    UA_Client_Service_Subscription_deleteSubscriptions(ctx->client, ctx->client->config.clientContext, request);

    ctx->active = false;
    ctx->subscriptionId = 0;
    ctx->client = NULL;
}

#ifndef OPCUA_SUBSCRIPTION_INTERFACE_H
#define OPCUA_SUBSCRIPTION_INTERFACE_H

#ifdef __cplusplus
extern "C" {
#endif

// Forward declaration for client type (opaque pointer)
typedef void UA_Client;

// Function declarations
unsigned int OpcUa_CreateSubscription(UA_Client* client, unsigned char priority, double publishingInterval, unsigned int* outError);
int OpcUa_UpdateSubscriptionInterval(unsigned int hdl, double newInterval);
void OpcUa_DeleteSubscription(unsigned int hdl);

#ifdef __cplusplus
}
#endif

#endif // OPCUA_SUBSCRIPTION_INTERFACE_H

FUNCTION_BLOCK FB_OpcUaCreateSubscription
VAR_INPUT
    Execute: BOOL := FALSE;
    ConnectionHdl: DWORD := 0;             // Handle to existing OPC UA client
    Priority: BYTE := 100;                 // Subscription priority (1–255)
    Timeout: TIME := T#5000ms;            // Max time to wait
    PublishingInterval: TIME := T#1000ms;  // Interval between publishes
END_VAR

VAR_OUTPUT
    Done: BOOL := FALSE;
    Busy: BOOL := FALSE;
    Error: BOOL := FALSE;
    ErrorID: DWORD := 0;
    SubscriptionHdl: DWORD := 0;          // Output handle for further use
END_VAR

VAR
    lastExecute: BOOL := FALSE;
    startTime: TIME := T#0ms;
    clientPtr: POINTER TO VOID := 0;
    intervalMs: REAL := 1000.0;
END_VAR

// Reset outputs
Done := FALSE;
Error := FALSE;

CASE TRUE OF
    NOT Execute:
        Busy := FALSE;
        IF lastExecute THEN
            // Optional: Clean up subscription here if needed
        END_IF;

    Execute AND NOT Busy:
        IF NOT lastExecute THEN
            startTime := TIME();
            Busy := TRUE;
            Done := FALSE;

            // Convert ConnectionHdl to client pointer (assumes opaque client handle mapping)
            clientPtr := ADR(ConnectionHdl); // Simplified example – actual system may map handles differently

            // Convert TIME to milliseconds (REAL)
            intervalMs := REAL(TIME_TO_DWORD(PublishingInterval)) / 10.0; // T#1s = 1000 ms

            // Call C function
            errorCode := OpcUa_CreateSubscription(clientPtr, Priority, intervalMs, ADR(ErrorID));
            IF errorCode <> 0 THEN
                Error := TRUE;
                ErrorID := errorCode;
                Busy := FALSE;
            ELSE
                SubscriptionHdl := errorCode; // Assuming returns handle on success
                Done := TRUE;
                Busy := FALSE;
            END_IF;

            lastExecute := TRUE;
        END_IF;

    Busy:
        // Check timeout
        IF (TIME() - startTime) > Timeout THEN
            Busy := FALSE;
            Error := TRUE;
            ErrorID := 16#8001; // Custom timeout error code
        END_IF;
END_CASE;
