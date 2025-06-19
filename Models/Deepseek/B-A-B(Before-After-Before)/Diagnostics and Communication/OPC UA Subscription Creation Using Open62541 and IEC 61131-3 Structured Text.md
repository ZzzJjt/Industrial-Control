/* opcua_subscription.h */
#include <open62541/client.h>
#include <open62541/client_subscriptions.h>

typedef struct {
    UA_UInt32 subId;
    UA_Double publishingInterval;
    UA_Byte priority;
    UA_StatusCode lastError;
} OPCUA_Subscription;

UA_StatusCode OPCUA_CreateSubscription(
    UA_Client* client,
    OPCUA_Subscription* sub,
    UA_Double interval,
    UA_Byte priority,
    UA_UInt32 timeoutMs);


/* plc_interface.c */
DLLEXPORT int OPCUA_EstablishSubscription(
    UA_UInt32 connectionHandle,
    UA_UInt32* subscriptionHandle,
    UA_Double* publishingInterval,
    UA_Byte priority,
    UA_UInt32 timeoutMs,
    UA_UInt32* errorCode)
{
    UA_Client* client = (UA_Client*)connectionHandle;
    OPCUA_Subscription sub = {0};
    
    UA_StatusCode ret = OPCUA_CreateSubscription(
        client, &sub, 
        *publishingInterval, 
        priority, 
        timeoutMs);
    
    *subscriptionHandle = sub.subId;
    *publishingInterval = sub.publishingInterval; // May be adjusted by server
    *errorCode = ret;
    
    return (ret == UA_STATUSCODE_GOOD) ? 1 : 0;
}

FUNCTION_BLOCK OPCUA_SubscriptionFB
VAR_INPUT
    Execute : BOOL;
    ConnectionHdl : DWORD;
    Priority : BYTE := 100;
    Timeout : TIME := T#5S;
    PublishingInterval : TIME := T#100MS;
END_VAR

VAR_OUTPUT
    Done : BOOL;
    Busy : BOOL;
    Error : BOOL;
    ErrorID : DWORD;
    SubscriptionHdl : DWORD;
END_VAR

VAR
    {attribute 'hidden'}
    internalState : INT := 0;
    {attribute 'hidden'}
    prevExecute : BOOL := FALSE;
    {attribute 'externallink':='opcua_client.dll'}
    {attribute 'functionprefix':=''}
    CreateSub : DWORD := 0;
    tempInterval : UDINT;
END_VAR

METHOD CreateSubscription : BOOL
VAR_TEMP
    executeEdge : BOOL;
    extResult : DWORD;
    actualInterval : TIME;
END_VAR

executeEdge := Execute AND NOT prevExecute;
prevExecute := Execute;

CASE internalState OF
    0: (* Idle *)
        IF executeEdge THEN
            Busy := TRUE;
            Done := FALSE;
            Error := FALSE;
            tempInterval := TIME_TO_UDINT(PublishingInterval);
            internalState := 10;
        END_IF
    
    10: (* Initiate subscription *)
        extResult := CreateSub(
            ConnectionHdl,
            ADR(SubscriptionHdl),
            ADR(tempInterval),
            Priority,
            TIME_TO_UDINT(Timeout),
            ADR(ErrorID));
        
        PublishingInterval := UDINT_TO_TIME(tempInterval);
        
        IF extResult <> 0 THEN
            internalState := 20; (* Success *)
        ELSE
            internalState := 30; (* Error *)
        END_IF
        
    20: (* Subscription successful *)
        Done := TRUE;
        Busy := FALSE;
        internalState := 0;
        RETURN TRUE;
        
    30: (* Error handling *)
        Error := TRUE;
        Busy := FALSE;
        internalState := 0;
END_CASE

RETURN FALSE;

PROGRAM MAIN
VAR
    subFB : OPCUA_SubscriptionFB;
    connFB : OPCUA_Connector;
    adaptInterval : BOOL := FALSE;
END_VAR

// First establish connection
connFB(Execute := TRUE, ServerUrl := 'opc.tcp://plc-server:4840');

// Then create subscription
subFB(
    Execute := connFB.Done,
    ConnectionHdl := connFB.ConnectionHdl,
    PublishingInterval := T#50MS);

// Runtime adjustment
IF adaptInterval AND NOT subFB.Busy THEN
    subFB.PublishingInterval := T#200MS;
    subFB.Execute := TRUE; // Re-trigger with new interval
END_IF

// In C module
UA_StatusCode OPCUA_DeleteSubscription(
    UA_Client* client,
    UA_UInt32 subId)
{
    return UA_Client_Subscriptions_deleteSingle(
        client, 
        UA_UInt32_to_NodeId(subId));
}

# CMakeLists.txt for cross-compilation
find_package(open62541 REQUIRED)
add_library(opcua_subscriptions SHARED 
    src/opcua_subscription.c 
    src/plc_interface.c)
target_link_libraries(opcua_subscriptions open62541::open62541)
set_target_properties(opcua_subscriptions PROPERTIES 
    SUFFIX ".library"  # CODESYS specific
    PREFIX "")
