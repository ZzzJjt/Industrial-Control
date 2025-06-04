#include <open62541/client_subscriptions.h>
#include <open62541/client_config_default.h>

typedef struct {
    UA_Client *client;
    UA_UInt32 subscriptionId;
    UA_Boolean subscriptionCreated;
    UA_StatusCode status;
} OPCUASubscription;

OPCUASubscription CreateOPCUASubscription(UA_Client *client, UA_Double publishingInterval) {
    OPCUASubscription result;
    result.client = client;
    result.subscriptionCreated = false;

    UA_CreateSubscriptionRequest request = UA_CreateSubscriptionRequest_default();
    request.requestedPublishingInterval = publishingInterval;

    UA_CreateSubscriptionResponse response =
        UA_Client_Subscriptions_create(client, request, NULL, NULL, NULL);

    if(response.responseHeader.serviceResult == UA_STATUSCODE_GOOD) {
        result.subscriptionCreated = true;
        result.subscriptionId = response.subscriptionId;
        result.status = UA_STATUSCODE_GOOD;
    } else {
        result.subscriptionId = 0;
        result.status = response.responseHeader.serviceResult;
    }

    return result;
}
