#include <open62541/client_config_default.h>
#include <open62541/client_highlevel.h>
#include <open62541/client_subscriptions.h>
#include <string.h>

// Create OPC UA Subscription
// Returns: 0 (Success), 1 (Invalid ConnectionHdl), 2 (Timeout), 3 (Server Unavailable), 4 (Invalid Parameters), 5 (Internal Error)
// Updates publishing_interval_ms with server-assigned value
int opc_ua_create_subscription(UA_Client *client, uint8_t priority, int timeout_ms, double *publishing_interval_ms, uint32_t *subscription_id) {
    if (!client) {
        return 1; // Invalid ConnectionHdl
    }
    if (*publishing_interval_ms <= 0 || priority > 255) {
        return 4; // Invalid Parameters
    }

    // Create subscription request
    UA_CreateSubscriptionRequest request = UA_CreateSubscriptionRequest_default();
    request.requestedPublishingInterval = *publishing_interval_ms;
    request.requestedPriority = priority;

    // Set timeout
    UA_ClientConfig *config = UA_Client_getConfig(client);
    uint32_t original_timeout = config->timeout;
    config->timeout = timeout_ms;

    // Create subscription
    UA_CreateSubscriptionResponse response = UA_Client_Subscriptions_create(client, request, NULL, NULL, NULL);
    config->timeout = original_timeout; // Restore timeout

    if (response.responseHeader.serviceResult == UA_STATUSCODE_GOOD) {
        *publishing_interval_ms = response.revisedPublishingInterval;
        *subscription_id = response.subscriptionId;
        return 0; // Success
    } else if (response.responseHeader.serviceResult == UA_STATUSCODE_BADTIMEOUT) {
        return 2; // Timeout
    } else if (response.responseHeader.serviceResult == UA_STATUSCODE_BADSERVERNOTFOUND ||
               response.responseHeader.serviceResult == UA_STATUSCODE_BADCONNECTIONCLOSED) {
        return 3; // Server Unavailable
    } else if (response.responseHeader.serviceResult == UA_STATUSCODE_BADINVALIDARGUMENT) {
        return 4; // Invalid Parameters
    } else {
        return 5; // Internal Error
    }
}

// Delete OPC UA Subscription
void opc_ua_delete_subscription(UA_Client *client, uint32_t subscription_id) {
    if (client && subscription_id != 0) {
        UA_Client_Subscriptions_deleteSingle(client, subscription_id);
    }
}
