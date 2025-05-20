#include <open62541/client_config_default.h>
#include <open62541/client_subscriptions.h>

/*
 * Function: opcua_subscribe
 * Purpose: Creates an OPC UA subscription on a connected client session
 * Parameters:
 *   client           - Pointer to UA_Client (session handle)
 *   pub_interval_ms  - Requested publishing interval (ms)
 *   priority         - Subscription priority (0-255)
 *   timeout_ms       - Operation timeout (ms)
 *   sub_id           - Output: Subscription ID
 *   actual_interval_ms - Output: Actual publishing interval
 * Returns:
 *   error_id - 0x0000 (success), 0x0001 (invalid session), 0x0002 (subscription failed),
 *              0x0003 (timeout), 0xFFFF (unknown error)
 */
unsigned long opcua_subscribe(void* client, unsigned long pub_interval_ms, unsigned char priority,
                             unsigned long timeout_ms, unsigned long* sub_id,
                             unsigned long* actual_interval_ms) {
    // Validate inputs
    if (!client) {
        return 0x0001; // Invalid session handle
    }
    if (pub_interval_ms == 0 || timeout_ms == 0) {
        return 0xFFFF; // Unknown error (invalid parameters)
    }

    // Configure subscription parameters
    UA_CreateSubscriptionRequest request = UA_CreateSubscriptionRequest_default();
    request.requestedPublishingInterval = (float)pub_interval_ms;
    request.requestedPriority = priority;

    // Create subscription
    UA_CreateSubscriptionResponse response = UA_Client_Subscriptions_create(
        (UA_Client*)client, request, NULL, NULL, NULL);

    // Handle response
    if (response.responseHeader.serviceResult == UA_STATUSCODE_GOOD) {
        *sub_id = response.subscriptionId;
        *actual_interval_ms = (unsigned long)response.revisedPublishingInterval;
        return 0x0000; // Success
    } else if (response.responseHeader.serviceResult == UA_STATUSCODE_BADTIMEOUT) {
        return 0x0003; // Timeout
    } else {
        return 0x0002; // Subscription failed
    }
}
