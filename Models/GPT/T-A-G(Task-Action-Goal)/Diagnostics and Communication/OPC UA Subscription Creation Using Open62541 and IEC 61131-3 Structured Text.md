(* C Layer OPC UA Subscription Stub using open62541 *)
// This is compiled and linked as part of the C integration layer

#include <open62541/client_subscriptions.h>

int opcua_create_subscription(unsigned int sessionHdl, double intervalMs, unsigned char priority, unsigned int timeoutMs, unsigned int* subscriptionHdlOut) {
    UA_Client *client = getClientFromHandle(sessionHdl); // implementation-specific
    if (!client) return -1;

    UA_CreateSubscriptionRequest request = UA_CreateSubscriptionRequest_default();
    request.requestedPublishingInterval = intervalMs;
    request.priority = priority;
    request.requestedLifetimeCount = 60;
    request.requestedMaxKeepAliveCount = 10;

    UA_CreateSubscriptionResponse response = UA_Client_Subscriptions_create(client, request, NULL, NULL, NULL);

    if (response.responseHeader.serviceResult != UA_STATUSCODE_GOOD)
        return (int)response.responseHeader.serviceResult;

    *subscriptionHdlOut = response.subscriptionId;
    return 0;
}


(* IEC 61131-3 Structured Text Function Block *)
FUNCTION_BLOCK FB_OPCUA_Subscribe
VAR_INPUT
    Execute            : BOOL;
    ConnectionHdl      : DWORD;
    Priority           : BYTE;
    Timeout            : TIME;
    PublishingInterval : TIME;
END_VAR
VAR_OUTPUT
    Done              : BOOL;
    Busy              : BOOL;
    Error             : BOOL;
    ErrorID           : DWORD;
    SubscriptionHdl   : DWORD;
END_VAR
VAR
    prevExecute       : BOOL := FALSE;
    state             : INT := 0;
    returnCode        : INT;
    intervalMs        : REAL;
END_VAR

(* Rising edge detection *)
IF Execute AND NOT prevExecute THEN
    Busy := TRUE;
    Done := FALSE;
    Error := FALSE;
    ErrorID := 0;
    state := 1;
END_IF
prevExecute := Execute;

CASE state OF

    1: // Initiate subscription
        intervalMs := REAL_TO_TIME(PublishingInterval) * 1000.0; // convert TIME to ms
        returnCode := opcua_create_subscription(
            ConnectionHdl,
            intervalMs,
            Priority,
            TO_UINT(Timeout),
            ADR(SubscriptionHdl));

        IF returnCode = 0 THEN
            Done := TRUE;
            Busy := FALSE;
            state := 0;
        ELSE
            Error := TRUE;
            ErrorID := DWORD(returnCode);
            Busy := FALSE;
            state := 0;
        END_IF
END_CASE
