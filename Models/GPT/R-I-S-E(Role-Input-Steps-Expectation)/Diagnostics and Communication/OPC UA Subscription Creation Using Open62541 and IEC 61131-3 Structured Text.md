#include <open62541/client.h>
#include <open62541/client_config_default.h>

typedef struct {
    UA_Client *client;
    UA_UInt32 subscriptionId;
    UA_Boolean active;
    UA_StatusCode lastStatus;
} OPCUA_SubscriptionHandle;

OPCUA_SubscriptionHandle g_subscription = {0};

UA_StatusCode opcua_create_subscription(
    UA_Client *client,
    UA_Byte priority,
    UA_Double publishingInterval,
    UA_UInt32 *out_subscriptionId
) {
    if(!client || !out_subscriptionId)
        return UA_STATUSCODE_BADINVALIDARGUMENT;

    UA_CreateSubscriptionRequest request = UA_CreateSubscriptionRequest_default();
    request.requestedPublishingInterval = publishingInterval;
    request.priority = priority;

    UA_CreateSubscriptionResponse response = UA_Client_Subscriptions_create(client, request, NULL, NULL, NULL);

    if(response.responseHeader.serviceResult == UA_STATUSCODE_GOOD) {
        *out_subscriptionId = response.subscriptionId;
        g_subscription.subscriptionId = response.subscriptionId;
        g_subscription.active = true;
        g_subscription.client = client;
        g_subscription.lastStatus = UA_STATUSCODE_GOOD;
        return UA_STATUSCODE_GOOD;
    }

    g_subscription.lastStatus = response.responseHeader.serviceResult;
    return response.responseHeader.serviceResult;
}

UA_StatusCode opcua_get_subscription_status() {
    return g_subscription.lastStatus;
}

UA_UInt32 opcua_get_subscription_id() {
    return g_subscription.subscriptionId;
}

#ifdef __cplusplus
extern "C" {
#endif

#include <stdint.h>

int opcua_create_subscription(void *client, uint8_t priority, double publishingInterval, uint32_t *subscriptionId);
int opcua_get_subscription_status();
uint32_t opcua_get_subscription_id();

#ifdef __cplusplus
}
#endif

FUNCTION_BLOCK FB_OPCUA_Subscribe
VAR_INPUT
    Execute           : BOOL;
    ConnectionHdl     : DWORD; // Assumes pointer to UA_Client*
    Priority          : BYTE;
    Timeout           : TIME;
END_VAR
VAR_IN_OUT
    PublishingInterval : TIME;
END_VAR
VAR_OUTPUT
    Done             : BOOL;
    Busy             : BOOL;
    Error            : BOOL;
    ErrorID          : DWORD;
    SubscriptionHdl  : DWORD;
END_VAR
VAR
    EdgeDetect       : R_TRIG;
    State            : INT;
    StartTime        : TIME;
    NowTime          : TIME;
END_VAR

// External declarations
FUNCTION opcua_create_subscription : DINT
VAR_INPUT
    client           : DWORD;
    priority         : BYTE;
    pubInterval      : REAL;
    subscriptionId   : POINTER TO DWORD;
END_VAR;

METHOD Main
EdgeDetect(CLK := Execute);

CASE State OF
0: // Idle
    Done := FALSE;
    Busy := FALSE;
    Error := FALSE;
    ErrorID := 0;
    IF EdgeDetect.Q THEN
        Busy := TRUE;
        StartTime := TIME();
        State := 1;
    END_IF

1: // Subscribing
    NowTime := TIME();
    IF (NowTime - StartTime) > Timeout THEN
        Error := TRUE;
        ErrorID := 16#DEAD0001; // Timeout
        Busy := FALSE;
        State := 0;
    ELSE
        VAR subID : DWORD;
        VAR pubReal : REAL;
        pubReal := TO_REAL(TO_DINT(PublishingInterval) / 1000); // ms → s
        ErrorID := opcua_create_subscription(
            ConnectionHdl,
            Priority,
            pubReal,
            ADR(subID)
        );
        IF ErrorID = 0 THEN
            SubscriptionHdl := subID;
            Done := TRUE;
            Busy := FALSE;
            State := 0;
        ELSE
            Error := TRUE;
            Busy := FALSE;
            State := 0;
        END_IF
    END_IF
END_CASE
