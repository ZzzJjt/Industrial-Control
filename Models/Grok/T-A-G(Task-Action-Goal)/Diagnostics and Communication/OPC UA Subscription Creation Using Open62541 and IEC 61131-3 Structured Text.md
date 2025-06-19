#include <open62541/client_config_default.h>
#include <open62541/client_highlevel.h>
#include <open62541/client_subscriptions.h>
#include <open62541/plugin/log_stdout.h>

typedef struct {
    UA_Client *client;
    UA_UInt32 subscriptionId;
    UA_Boolean isActive;
    UA_UInt32 errorCode;
} OPCUA_Subscription;

OPCUA_Subscription* opcua_subscription_init() {
    OPCUA_Subscription *sub = (OPCUA_Subscription*)UA_calloc(1, sizeof(OPCUA_Subscription));
    if (!sub) return NULL;
    
    sub->client = NULL;
    sub->subscriptionId = 0;
    sub->isActive = UA_FALSE;
    sub->errorCode = 0;
    return sub;
}

UA_StatusCode opcua_subscription_create(
    OPCUA_Subscription *sub,
    UA_Client *client,
    UA_Double publishingIntervalMs,
    UA_Byte priority,
    UA_UInt32 timeoutMs,
    UA_UInt32 *subscriptionId
) {
    if (!sub || !client || !subscriptionId || publishingIntervalMs <= 0) {
        sub->errorCode = 0x80040000; // Invalid parameters
        return UA_STATUSCODE_BADINVALIDARGUMENT;
    }

    sub->client = client;
    UA_CreateSubscriptionRequest request = UA_CreateSubscriptionRequest_default();
    request.requestedPublishingInterval = publishingIntervalMs;
    request.requestedPriority = priority;
    request.requestedMaxKeepAliveCount = 10;
    request.requestedLifetimeCount = 30;

    UA_CreateSubscriptionResponse response = UA_Client_Subscriptions_create(client, request, NULL, NULL, NULL);
    UA_StatusCode status = response.responseHeader.serviceResult;

    if (status == UA_STATUSCODE_GOOD) {
        sub->subscriptionId = response.subscriptionId;
        sub->isActive = UA_TRUE;
        sub->errorCode = 0;
        *subscriptionId = sub->subscriptionId;
    } else {
        sub->subscriptionId = 0;
        sub->isActive = UA_FALSE;
        sub->errorCode = status;
        *subscriptionId = 0;
    }

    UA_CreateSubscriptionResponse_deleteMembers(&response);
    return status;
}

UA_StatusCode opcua_subscription_update_interval(
    OPCUA_Subscription *sub,
    UA_Double publishingIntervalMs
) {
    if (!sub || !sub->isActive || !sub->client) {
        sub->errorCode = 0x80050000; // Invalid state
        return UA_STATUSCODE_BADINVALIDSTATE;
    }

    UA_ModifySubscriptionRequest request;
    UA_ModifySubscriptionRequest_init(&request);
    request.subscriptionId = sub->subscriptionId;
    request.requestedPublishingInterval = publishingIntervalMs;
    request.requestedMaxKeepAliveCount = 10;
    request.requestedLifetimeCount = 30;

    UA_ModifySubscriptionResponse response = UA_Client_Subscriptions_modify(sub->client, request);
    UA_StatusCode status = response.responseHeader.serviceResult;

    if (status != UA_STATUSCODE_GOOD) {
        sub->errorCode = status;
    } else {
        sub->errorCode = 0;
    }

    UA_ModifySubscriptionResponse_deleteMembers(&response);
    return status;
}

void opcua_subscription_cleanup(OPCUA_Subscription *sub) {
    if (!sub) return;

    if (sub->isActive && sub->client) {
        UA_Client_Subscriptions_deleteSingle(sub->client, sub->subscriptionId);
    }

    UA_free(sub);
}

UA_UInt32 opcua_subscription_get_error(OPCUA_Subscription *sub) {
    return sub ? sub->errorCode : 0x80060000; // Null context
}

UA_Boolean opcua_subscription_is_active(OPCUA_Subscription *sub) {
    return sub ? sub->isActive : UA_FALSE;
}
