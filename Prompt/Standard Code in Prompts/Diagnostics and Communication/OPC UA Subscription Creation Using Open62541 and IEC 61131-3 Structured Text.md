```
#include <open62541/client_high_level_common.h>
#include <open62541/client_subscription.h>
#include <stdlib.h>

// Function prototype for the callback to handle subscription data notifications
static void subscription_data_notification_callback(UA_Client *client,
                                                    const UA_DataNotification *notification,
                                                    void *userContext);

// Function to create a subscription
UA_StatusCode createSubscription(UA_Client *client,
                                 UA_UInt32 *subscriptionHdl,
                                 UA_Byte priority,
                                 UA_TimeDuration publishingInterval,
                                 UA_UInt32 *revisedPublishingInterval,
                                 UA_UInt32 maxNotificationsPerPublish,
                                 UA_Double publishingEnabledStartDelay,
                                 UA_Boolean keepAliveCount,
                                 UA_Boolean maxKeepAliveCount,
                                 UA_Boolean maxLifetimeCount,
                                 UA_Boolean maxNodesPerRead,
                                 UA_Boolean maxNodesPerHistoryRead,
                                 UA_Boolean maxNotificationsPerCall,
                                 UA_UInt32 *revisedMaxNotificationsPerPublish,
                                 UA_Byte *revisedPriority) {
    // Create subscription
    UA_StatusCode retval = UA_Client_CreateSubscription(
        client,
        publishingInterval,
        maxNotificationsPerPublish,
        publishingEnabledStartDelay,
        keepAliveCount,
        maxKeepAliveCount,
        maxLifetimeCount,
        maxNodesPerRead,
        maxNodesPerHistoryRead,
        maxNotificationsPerCall,
        subscriptionHdl,
        revisedPublishingInterval,
        revisedMaxNotificationsPerPublish,
        revisedPriority,
        subscription_data_notification_callback,
        NULL); // User context is NULL in this example

    return retval;
}

// Callback function for handling subscription data notifications
static void subscription_data_notification_callback(UA_Client *client,
                                                    const UA_DataNotification *notification,
                                                    void *userContext) {
    // Handle data notification here
    // This is a placeholder for actual implementation
    (void)client; // Suppress warning
    (void)notification; // Suppress warning
    (void)userContext; // Suppress warning
}
```
```
FUNCTION_BLOCK OPC_UA_SUBSCRIPTION_CREATION
VAR_INPUT
    Exec : BOOL;
    ConnectionHandle : DWORD;
    Priority : BYTE;
    Timeout : TIME;
VAR_OUTPUT
    Done : BOOL;
    Busy : BOOL;
    Error : BOOL;
    ErrorID : DWORD;
    SubscriptionHdl : DWORD;
VAR_IN_OUT
    PublishingInterval : TIME;
VAR
    CClient : ANY; // Placeholder for the C client object
    CConnectionHandle : POINTER TO ANY; // Pointer to the connection handle
    CPriority : UINT;
    CTimeout : UINT;
    CPublishingInterval : UINT;
END_VAR

// Initialize pointers and conversions
CConnectionHandle := PTR_CONNECTION(ConnectionHandle);
CPriority := Priority;
CTimeout := TIMEOUT;
CPublishingInterval := PUBLISHING_INTERVAL;

IF Exec THEN
    // Convert time to milliseconds for C library
    CPublishingInterval := PublishingInterval / T#1ms;
    
    // Call the C function to create the subscription
    ErrorID := CALL_C_FUNC(CreateSubscription,
                           CConnectionHandle,
                           CPriority,
                           CPublishingInterval,
                           SubscriptionHdl);
    
    // Check the result
    IF ErrorID = 0 THEN
        Done := TRUE;
        Busy := FALSE;
        Error := FALSE;
    ELSE
        Done := FALSE;
        Busy := FALSE;
        Error := TRUE;
    END_IF
ELSE
    Done := FALSE;
    Busy := TRUE;
    Error := FALSE;
END_IF

// Example pseudo-function for demonstration
PROCEDURE CALL_C_FUNC(FuncName : STRING; ConnHandle : POINTER TO ANY; Priority : UINT; Timeout : UINT; SubscriptionHdl : REFERENCE TO DWORD) : DWORD
    // Calls the specified C function with the provided arguments and returns the error ID
    RETURN 0; // Placeholder for the actual error ID
END_PROC
END_FUNCTION_BLOCK

PROGRAM OPC_UA_CLIENT_PROGRAM
VAR
    ConnectionHandle : DWORD := 0x12345678; // Example connection handle
    Priority : BYTE := 1; // Example priority level
    Timeout : TIME := T#5s; // Example timeout
    PublishingInterval : TIME := T#1000ms; // Initial publishing interval
    Done : BOOL;
    Busy : BOOL;
    Error : BOOL;
    ErrorID : DWORD;
    SubscriptionHdl : DWORD;
BEGIN
    OPC_UA_SUBSCRIPTION_CREATION(Exec := TRUE,
                                 ConnectionHandle,
                                 Priority,
                                 Timeout,
                                 Done,
                                 Busy,
                                 Error,
                                 ErrorID,
                                 SubscriptionHdl,
                                 PublishingInterval);
    
    // Use the outputs to control further actions
    IF Done THEN
        // Subscription created successfully
        // Further processing goes here
    ELSIF Error THEN
        // Handle error condition
        WRITE_LOG(ErrorID);
    ELSE
        // Busy state, wait for completion
    END_IF
END
```
