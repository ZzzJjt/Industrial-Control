#include <open62541/client_config_default.h>
#include <open62541/client_highlevel.h>
#include <open62541/client_subscriptions.h>
#include <string.h>
#include <stdint.h>

// Client context
static UA_Client* client = NULL;
static UA_StatusCode last_status = UA_STATUSCODE_GOOD;
static uint32_t last_subscription_id = 0;

// Initialize subscription
int32_t opcua_create_subscription(uint32_t connection_hdl, uint8_t priority, float publishing_interval_ms, uint32_t* subscription_hdl) {
    if (client == NULL || connection_hdl != (uint32_t)client) {
        return UA_STATUSCODE_BADSESSIONINVALID;
    }

    UA_CreateSubscriptionRequest request = UA_CreateSubscriptionRequest_default();
    request.requestedPublishingInterval = publishing_interval_ms;
    request.priority = priority;

    UA_CreateSubscriptionResponse response = UA_Client_Subscriptions_create(client, request, NULL, NULL, NULL);
    last_status = response.responseHeader.serviceResult;
    if (last_status != UA_STATUSCODE_GOOD) {
        return last_status;
    }

    last_subscription_id = response.subscriptionId;
    *subscription_hdl = response.subscriptionId;
    return UA_STATUSCODE_GOOD;
}

// Delete subscription
int32_t opcua_delete_subscription(uint32_t subscription_hdl) {
    if (client == NULL || subscription_hdl != last_subscription_id) {
        return UA_STATUSCODE_BADSUBSCRIPTIONIDINVALID;
    }

    last_status = UA_Client_Subscriptions_deleteSingle(client, subscription_hdl);
    if (last_status == UA_STATUSCODE_GOOD) {
        last_subscription_id = 0;
    }
    return last_status;
}

// Get subscription status
int32_t opcua_get_subscription_status(uint32_t subscription_hdl) {
    if (client == NULL || subscription_hdl != last_subscription_id) {
        return UA_STATUSCODE_BADSUBSCRIPTIONIDINVALID;
    }
    return last_status;
}

// Set client (called by connection function block)
void opcua_set_client(void* client_ptr) {
    client = (UA_Client*)client_ptr;
}
