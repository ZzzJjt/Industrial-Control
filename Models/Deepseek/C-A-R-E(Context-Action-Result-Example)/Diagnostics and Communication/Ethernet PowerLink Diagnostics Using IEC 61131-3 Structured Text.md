FUNCTION_BLOCK POWERLINK_DIAGNOSTIC
VAR_INPUT
    MN_Interface : REFERENCE TO MANAGING_NODE_INTERFACE; // Reference to the Managing Node Interface
END_VAR

VAR_OUTPUT
    DiagnosticsUpdated : BOOL; // Indicates if diagnostics have been updated
    CommunicationStatus : ARRAY[1..10] OF STRING[50]; // Communication status of each node
    ErrorCodes : ARRAY[1..10] OF INT; // Error codes for each node
    NodeHealth : ARRAY[1..10] OF STRING[50]; // Health status of each node
END_VAR

VAR
    LastCommunicationStatus : ARRAY[1..10] OF STRING[50];
    LastErrorCodes : ARRAY[1..10] OF INT;
    LastNodeHealth : ARRAY[1..10] OF STRING[50];
    Timer : TON; // Timer for cyclic updates
END_VAR

// Method to retrieve diagnostic data from a single node
METHOD RetrieveDiagnosticData : BOOL
VAR_INPUT
    NodeID : INT;
END_VAR
VAR
    TempCommStatus : STRING[50];
    TempErrorCode : INT;
    TempNodeHealth : STRING[50];
END_VAR
    // Simulate retrieval of diagnostic data from the node
    // In a real system, this would involve actual communication with the node
    CASE NodeID OF
        1:
            TempCommStatus := 'Connected';
            TempErrorCode := 0;
            TempNodeHealth := 'Healthy';
        2:
            TempCommStatus := 'Disconnected';
            TempErrorCode := 0x01;
            TempNodeHealth := 'Unhealthy';
        3:
            TempCommStatus := 'Connected';
            TempErrorCode := 0;
            TempNodeHealth := 'Healthy';
        4:
            TempCommStatus := 'Connected';
            TempErrorCode := 0;
            TempNodeHealth := 'Healthy';
        5:
            TempCommStatus := 'Connected';
            TempErrorCode := 0;
            TempNodeHealth := 'Healthy';
        6:
            TempCommStatus := 'Connected';
            TempErrorCode := 0;
            TempNodeHealth := 'Healthy';
        7:
            TempCommStatus := 'Connected';
            TempErrorCode := 0;
            TempNodeHealth := 'Healthy';
        8:
            TempCommStatus := 'Connected';
            TempErrorCode := 0;
            TempNodeHealth := 'Healthy';
        9:
            TempCommStatus := 'Connected';
            TempErrorCode := 0;
            TempNodeHealth := 'Healthy';
        10:
            TempCommStatus := 'Connected';
            TempErrorCode := 0;
            TempNodeHealth := 'Healthy';
        ELSE
            RETURN FALSE; // Invalid NodeID
    END_CASE;

    CommunicationStatus[NodeID] := TempCommStatus;
    ErrorCodes[NodeID] := TempErrorCode;
    NodeHealth[NodeID] := TempNodeHealth;
    RETURN TRUE;
END_METHOD

// Main execution logic
DiagnosticsUpdated := FALSE;

// Cyclic update every 5 seconds
Timer(IN := TRUE, PT := T#5s);
IF Timer.Q THEN
    Timer(IN := FALSE); // Reset the timer
    FOR i := 1 TO SIZEOF(CommunicationStatus, 1) DO
        IF NOT RetrieveDiagnosticData(i) THEN
            // Handle error: invalid NodeID or other issues
            PRINTF('Failed to retrieve diagnostic data for Node %d\n', i);
            Continue;
        END_IF;
        IF CommunicationStatus[i] <> LastCommunicationStatus[i] OR
           ErrorCodes[i] <> LastErrorCodes[i] OR
           NodeHealth[i] <> LastNodeHealth[i] THEN
            DiagnosticsUpdated := TRUE;
            LastCommunicationStatus[i] := CommunicationStatus[i];
            LastErrorCodes[i] := ErrorCodes[i];
            LastNodeHealth[i] := NodeHealth[i];
        END_IF;
    END_FOR;
END_IF;



