FUNCTION_BLOCK OPC_UA_SUBSCRIPTION
VAR_INPUT
    Execute : BOOL;                  // Starts the subscription creation
    ConnectionHdl : DWORD;           // OPC UA session handle
    Priority : BYTE;                 // Defines subscription priority
    Timeout : TIME;                  // Maximum wait time
    PublishingInterval : TIME;       // Defines or updates the subscription rate
END_VAR

VAR_OUTPUT
    Done : BOOL;                     // TRUE when the subscription is successfully created
    Busy : BOOL;                     // TRUE while the process is ongoing
    Error : BOOL;                    // TRUE if a failure occurs
    ErrorID : DWORD;                 // Returns diagnostic error code
    SubscriptionHdl : DWORD;         // Outputs the subscription handle
END_VAR

VAR
    lastExecute : BOOL := FALSE;     // Last state of the Execute input
    opcuaSubscriptionHandle : REFERENCE; // Handle to the OPC UA subscription
    subscribeAttempted : BOOL := FALSE; // Flag to indicate if subscription has been attempted
    subscribeSuccess : BOOL := FALSE; // Flag to indicate if subscription was successful
    subscribeError : DWORD := 0;     // Error code for subscription attempt
END_VAR

METHOD CreateSubscription : BOOL
VAR_INPUT
    connectionHdl : DWORD;
    publishingInterval : DOUBLE;
    priority : BYTE;
END_VAR
VAR
    success : BOOL := FALSE;
BEGIN
    IF opcua_subscription_create_c(connectionHdl, publishingInterval, priority, ADR(SubscriptionHdl), ADR(subscribeError)) THEN
        success := TRUE;
    END_IF;
    RETURN success;
END_METHOD

METHOD CleanupSubscription : BOOL
BEGIN
    opcua_subscription_cleanup_c(SubscriptionHdl);
    RETURN TRUE;
END_METHOD

METHOD Execute : BOOL
BEGIN
    IF R_TRIG(lastExecute, Execute).Q THEN
        IF NOT subscribeAttempted THEN
            Busy := TRUE;
            Done := FALSE;
            Error := FALSE;
            subscribeSuccess := CreateSubscription(ConnectionHdl, REAL_TO_LREAL(PublishingInterval) / 1000.0, Priority);
            subscribeAttempted := TRUE;
            IF subscribeSuccess THEN
                Done := TRUE;
                Busy := FALSE;
            ELSE
                Error := TRUE;
                ErrorID := subscribeError;
                Busy := FALSE;
            END_IF;
        END_IF;
    ELSIF NOT Execute THEN
        lastExecute := FALSE;
        subscribeAttempted := FALSE;
        subscribeSuccess := FALSE;
        subscribeError := 0;
        Done := FALSE;
        Busy := FALSE;
        Error := FALSE;
        ErrorID := 0;
        SubscriptionHdl := 0;
        CleanupSubscription();
    END_IF;

    lastExecute := Execute;
    RETURN TRUE;
END_METHOD

END_FUNCTION_BLOCK



