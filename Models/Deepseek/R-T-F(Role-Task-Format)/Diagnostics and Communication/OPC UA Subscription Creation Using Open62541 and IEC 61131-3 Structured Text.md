#include <open62541/client.h>
#include <open62541/plugin/log_stdout.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct {
    UA_Client *client;
    UA_Subscription *subscription;
    DWORD connectionHdl;
    BYTE priority;
    TIME timeout;
    TIME publishingInterval;
    int done;
    int busy;
    int error;
    unsigned long errorId;
    DWORD subscriptionHdl;
} OpcUaSubscriptionClient;

OpcUaSubscriptionClient* opcua_subscription_client_create(DWORD connectionHdl, BYTE priority, TIME timeout, TIME publishingInterval) {
    OpcUaSubscriptionClient* clientData = malloc(sizeof(OpcUaSubscriptionClient));
    if (!clientData) return NULL;
    clientData->client = NULL; // Assume client handle is managed externally
    clientData->connectionHdl = connectionHdl;
    clientData->priority = priority;
    clientData->timeout = timeout;
    clientData->publishingInterval = publishingInterval;
    clientData->done = 0;
    clientData->busy = 0;
    clientData->error = 0;
    clientData->errorId = 0;
    clientData->subscriptionHdl = 0;
    return clientData;
}

void opcua_subscription_client_destroy(OpcUaSubscriptionClient* clientData) {
    if (clientData) {
        if (clientData->subscription) {
            UA_Client_deleteSubscription(clientData->client, clientData->subscription);
        }
        free(clientData);
    }
}

int opcua_subscription_client_create_subscription(OpcUaSubscriptionClient* clientData) {
    if (!clientData || !clientData->client) return -1;
    clientData->busy = 1;
    clientData->done = 0;
    clientData->error = 0;
    clientData->errorId = 0;

    UA_CreateSubscriptionRequest request = UA_CreateSubscriptionRequest_default();
    request.requestedPublishingInterval = clientData->publishingInterval / 1000000.0; // Convert from us to ms
    request.maxNotificationsPerPublish = 1000;
    request.publishingEnabled = true;
    request.priority = clientData->priority;

    UA_CreateSubscriptionResponse response;
    UA_Client_Service_createSubscription(clientData->client, &request, &response);

    if (response.responseHeader.serviceResult != UA_STATUSCODE_GOOD) {
        clientData->busy = 0;
        clientData->error = 1;
        clientData->errorId = response.responseHeader.serviceResult;
        UA_CreateSubscriptionResponse_clear(&response);
        return -1;
    }

    clientData->subscription = response.subscriptionId;
    clientData->subscriptionHdl = (DWORD)response.subscriptionId;
    clientData->busy = 0;
    clientData->done = 1;
    UA_CreateSubscriptionResponse_clear(&response);
    return 0;
}

UA_Client* opcua_subscription_client_get_client(OpcUaSubscriptionClient* clientData) {
    if (!clientData) return NULL;
    return clientData->client;
}

void opcua_subscription_client_set_client(OpcUaSubscriptionClient* clientData, UA_Client* client) {
    if (clientData) {
        clientData->client = client;
    }
}


FUNCTION_BLOCK OPC_UA_SUBSCRIPTION_CLIENT
VAR_INPUT
    Execute : BOOL;             // Triggers the subscription creation
    ConnectionHdl : DWORD;       // OPC UA session handle
    Priority : BYTE;              // Subscription priority level
    Timeout : TIME;              // Max wait duration for the operation
    PublishingInterval : TIME;   // The data update interval, dynamically adjustable
END_VAR

VAR_OUTPUT
    Done : BOOL;                 // Indicates successful subscription creation
    Busy : BOOL;                 // TRUE while the operation is in progress
    Error : BOOL;                // Set if an error occurs
    ErrorID : DWORD;             // Provides a specific error code
    SubscriptionHdl : DWORD;     // The handle of the newly created subscription
END_VAR

VAR
    lastExecute : BOOL := FALSE;
    clientHandle : POINTER TO ANY;
    startTime : TIME_OF_DAY;
    connected : BOOL := FALSE;
END_VAR

// Method to initialize the OPC UA subscription client
METHOD InitializeClient : BOOL
VAR_EXTERNAL
    opcua_subscription_client_create : FUNCTION(connectionHdl : DWORD; priority : BYTE; timeout : TIME; publishingInterval : TIME) : POINTER TO ANY;
END_VAR
BEGIN
    clientHandle := opcua_subscription_client_create(ConnectionHdl, Priority, Timeout, PublishingInterval);
    IF clientHandle IS_NULL THEN
        Error := TRUE;
        ErrorID := 1; // Generic error ID for initialization failure
        RETURN FALSE;
    END_IF;
    RETURN TRUE;
END_METHOD

// Method to set the OPC UA client
METHOD SetClient : BOOL
VAR_EXTERNAL
    opcua_subscription_client_set_client : PROCEDURE(clientHandle : POINTER TO ANY; client : POINTER TO ANY);
END_VAR
VAR_INPUT
    client : POINTER TO ANY;
END_VAR
BEGIN
    opcua_subscription_client_set_client(clientHandle, client);
    RETURN TRUE;
END_METHOD

// Method to get the OPC UA client
METHOD GetClient : POINTER TO ANY
VAR_EXTERNAL
    opcua_subscription_client_get_client : FUNCTION(clientHandle : POINTER TO ANY) : POINTER TO ANY;
END_VAR
BEGIN
    RETURN opcua_subscription_client_get_client(clientHandle);
END_METHOD

// Method to create the OPC UA subscription
METHOD CreateSubscription : BOOL
VAR_EXTERNAL
    opcua_subscription_client_create_subscription : FUNCTION(clientHandle : POINTER TO ANY) : INT;
END_VAR
BEGIN
    IF opcua_subscription_client_create_subscription(clientHandle) <> 0 THEN
        Error := TRUE;
        ErrorID := 2; // Generic error ID for subscription creation failure
        RETURN FALSE;
    END_IF;
    RETURN TRUE;
END_METHOD

// Method to destroy the OPC UA subscription client
METHOD DestroyClient : BOOL
VAR_EXTERNAL
    opcua_subscription_client_destroy : PROCEDURE(clientHandle : POINTER TO ANY);
END_VAR
BEGIN
    opcua_subscription_client_destroy(clientHandle);
    RETURN TRUE;
END_METHOD

// Main execution logic
METHOD Execute : BOOL
VAR
    currentTime : TIME_OF_DAY;
    uaClient : POINTER TO ANY;
BEGIN
    // Get current time
    currentTime := CURRENT_TIME_OF_DAY();

    // Rising edge detection on Execute
    IF Execute AND NOT lastExecute THEN
        lastExecute := TRUE;
        Done := FALSE;
        Busy := TRUE;
        Error := FALSE;
        ErrorID := 0;
        SubscriptionHdl := 0;

        // Initialize the client
        IF NOT InitializeClient() THEN
            Busy := FALSE;
            RETURN FALSE;
        END_IF;

        // Start the timer
        startTime := currentTime;

        // Set the OPC UA client (assuming this is provided externally)
        uaClient := ...; // Provide the actual UA_Client pointer here
        IF NOT SetClient(uaClient) THEN
            Busy := FALSE;
            RETURN FALSE;
        END_IF;

        // Attempt to create the subscription
        IF NOT CreateSubscription() THEN
            Busy := FALSE;
            RETURN FALSE;
        END_IF;

        // Check subscription status
        SubscriptionHdl := opcua_subscription_client_get_subscription_handle(clientHandle);
        IF SubscriptionHdl <> 0 THEN
            Done := TRUE;
            Busy := FALSE;
        ELSE
            // Check for timeout
            IF (currentTime - startTime) >= Timeout THEN
                Busy := FALSE;
                Error := TRUE;
                ErrorID := 3; // Timeout error ID
            END_IF;
        END_IF;
    ELSIF NOT Execute THEN
        lastExecute := FALSE;
    END_IF;

    RETURN TRUE;
END_METHOD

// Helper method to get the subscription handle
METHOD GetSubscriptionHandle : DWORD
VAR_EXTERNAL
    opcua_subscription_client_get_subscription_handle : FUNCTION(clientHandle : POINTER TO ANY) : DWORD;
END_VAR
BEGIN
    RETURN opcua_subscription_client_get_subscription_handle(clientHandle);
END_METHOD

END_FUNCTION_BLOCK

PROGRAM MAIN
VAR
    opcUaSubscriptionFB : OPC_UA_SUBSCRIPTION_CLIENT;
    execute : BOOL := FALSE;
    connectionHdl : DWORD := 12345; // Example connection handle
    priority : BYTE := 10;
    timeout : TIME := T#10s;
    publishingInterval : TIME := T#500ms;
    success : BOOL;
END_VAR

// Simulate enabling the OPC UA subscription client
execute := TRUE;

// Call the function block cyclically
WHILE TRUE DO
    opcUaSubscriptionFB.Execute := execute;
    opcUaSubscriptionFB.ConnectionHdl := connectionHdl;
    opcUaSubscriptionFB.Priority := priority;
    opcUaSubscriptionFB.Timeout := timeout;
    opcUaSubscriptionFB.PublishingInterval := publishingInterval;
    success := opcUaSubscriptionFB.Execute();

    // Check outputs
    IF success THEN
        IF opcUaSubscriptionFB.Done THEN
            (* PRINTF("Subscription Created Successfully, Handle: %d", opcUaSubscriptionFB.SubscriptionHdl); *)
            execute := FALSE; // Stop further execution attempts
        ELSIF opcUaSubscriptionFB.Busy THEN
            (* PRINTF("Creating Subscription..."); *)
        ELSIF opcUaSubscriptionFB.Error THEN
            (* PRINTF("Error: %d", opcUaSubscriptionFB.ErrorID); *)
            execute := FALSE; // Stop further execution attempts
        END_IF;
    END_IF;

    // Simulate periodic calls (e.g., every 100 ms)
    SLEEP(T#100ms);
END_WHILE;
END_PROGRAM
