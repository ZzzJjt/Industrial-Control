#ifndef OPCUA_SUBSCRIPTION_H
#define OPCUA_SUBSCRIPTION_H

#include <stdint.h>
#include <stdbool.h>

#ifdef __cplusplus
extern "C" {
#endif

typedef struct {
    bool done;
    bool busy;
    bool error;
    uint32_t errorId;
    uint32_t subscriptionId;
} OPCUA_SubscriptionStatus;

void OPCUA_SubscriptionCreate(
    uint32_t connectionHdl,
    uint8_t priority,
    uint32_t timeoutMs,
    uint32_t* publishingIntervalMs, // INOUT
    OPCUA_SubscriptionStatus* status
);

#ifdef __cplusplus
}
#endif

#endif

#include "opcua_subscription.h"
#include <open62541/client_subscriptions.h>

extern UA_Client* OPCUA_GetClientFromHandle(uint32_t hdl); // Assume session handle is mapped

void OPCUA_SubscriptionCreate(
    uint32_t connectionHdl,
    uint8_t priority,
    uint32_t timeoutMs,
    uint32_t* publishingIntervalMs,
    OPCUA_SubscriptionStatus* status)
{
    status->done = false;
    status->busy = true;
    status->error = false;
    status->errorId = 0;
    status->subscriptionId = 0;

    UA_Client* client = OPCUA_GetClientFromHandle(connectionHdl);
    if (client == NULL) {
        status->error = true;
        status->errorId = 0xDEAD0001; // Invalid handle
        status->busy = false;
        return;
    }

    UA_CreateSubscriptionRequest request = UA_CreateSubscriptionRequest_default();
    request.requestedPublishingInterval = (UA_Double)(*publishingIntervalMs);
    request.requestedLifetimeCount = 1000;
    request.requestedMaxKeepAliveCount = 10;
    request.priority = priority;

    UA_CreateSubscriptionResponse response = UA_Client_Subscriptions_create(client, request, NULL, NULL, NULL);

    if (response.responseHeader.serviceResult == UA_STATUSCODE_GOOD) {
        status->done = true;
        status->subscriptionId = response.subscriptionId;
    } else {
        status->error = true;
        status->errorId = response.responseHeader.serviceResult;
    }

    status->busy = false;
}

FUNCTION_BLOCK FB_OPCUA_SubscriptionCreate
VAR_INPUT
    Execute             : BOOL;
    ConnectionHdl       : DWORD;
    Priority            : BYTE;
    Timeout             : TIME;
    PublishingInterval  : IN_OUT TIME;
END_VAR

VAR_OUTPUT
    Done                : BOOL;
    Busy                : BOOL;
    Error               : BOOL;
    ErrorID             : DWORD;
    SubscriptionHdl     : DWORD;
END_VAR

VAR
    RisingEdge          : R_TRIG;
    CallInProgress      : BOOL := FALSE;
    CStatus             : OPCUA_SubscriptionStatus;
    PublishingIntervalMs: DWORD;
END_VAR

(* Rising-edge trigger *)
RisingEdge(CLK := Execute);

IF RisingEdge.Q THEN
    Busy := TRUE;
    Done := FALSE;
    Error := FALSE;
    ErrorID := 0;
    SubscriptionHdl := 0;
    CallInProgress := TRUE;

    PublishingIntervalMs := TO_DWORD(PublishingInterval) / 1000; // Convert TIME to ms

    OPCUA_SubscriptionCreate(
        connectionHdl := ConnectionHdl,
        priority := Priority,
        timeoutMs := TO_DWORD(Timeout) / 1000,
        publishingIntervalMs := ADR(PublishingIntervalMs),
        status := ADR(CStatus)
    );
END_IF

IF CallInProgress THEN
    Busy := CStatus.busy;
    Done := CStatus.done;
    Error := CStatus.error;
    ErrorID := CStatus.errorId;
    SubscriptionHdl := CStatus.subscriptionId;

    IF Done OR Error THEN
        PublishingInterval := T#1MS * PublishingIntervalMs;
        CallInProgress := FALSE;
    END_IF
END_IF
