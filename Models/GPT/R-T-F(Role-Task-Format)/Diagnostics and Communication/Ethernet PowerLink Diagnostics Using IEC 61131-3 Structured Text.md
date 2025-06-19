FUNCTION_BLOCK FB_PowerLinkDiagnostics
VAR_INPUT
    Enable          : BOOL;       // Enables the diagnostic polling cycle
    Trigger         : BOOL;       // Rising edge triggers diagnostic request
    NodeAddress     : INT;        // Target node ID on the PowerLink network
END_VAR

VAR_OUTPUT
    CommStatus      : BOOL;       // TRUE = communication OK
    ErrorCode       : INT;        // Last error code (0 = no error)
    NodeHealth      : STRING(20); // Status: 'OK', 'Warning', 'Fault', 'No Response'
END_VAR

VAR
    PrevTrigger     : BOOL := FALSE;         // Rising edge detection
    PollingActive   : BOOL := FALSE;         // Internal status
    ResponseTimeout : TON;                   // Timeout timer
    RetryCounter    : INT := 0;              // Retry attempts
    MaxRetries      : INT := 3;              // Retry limit
    RawDiagData     : BYTE;                  // Simulated response (e.g., bitfield from MN)
END_VAR

// Simulated helper function - Replace with actual interface to MN
FUNCTION_BLOCK FB_ReadPowerLinkDiagnostics
VAR_INPUT
    NodeID  : INT;
    Start   : BOOL;
END_VAR
VAR_OUTPUT
    Done    : BOOL;
    Error   : BOOL;
    Data    : BYTE;   // 8-bit diagnostic result
END_VAR
VAR
    InternalTimer : TON;
END_VAR

InternalTimer(IN := Start, PT := T#300ms);
IF InternalTimer.Q THEN
    Done := TRUE;
    Error := FALSE;
    Data := 2#00000001; // Example diagnostic bitmask: OK
ELSE
    Done := FALSE;
    Error := FALSE;
END_IF

// Trigger Rising Edge
IF Enable AND Trigger AND NOT PrevTrigger THEN
    PollingActive := TRUE;
    RetryCounter := 0;
    ResponseTimeout(IN := FALSE); // Reset
END_IF
PrevTrigger := Trigger;

// Start Polling
IF PollingActive THEN
    FB_ReadPowerLinkDiagnostics(NodeID := NodeAddress, Start := TRUE);

    // If data received successfully
    IF FB_ReadPowerLinkDiagnostics.Done THEN
        RawDiagData := FB_ReadPowerLinkDiagnostics.Data;
        CommStatus := TRUE;
        ErrorCode := 0;

        // Interpret diagnostic byte
        CASE RawDiagData OF
            2#00000001: NodeHealth := 'OK';
            2#00000010: NodeHealth := 'Warning';
            2#00000100: NodeHealth := 'Fault';
            ELSE       NodeHealth := 'Unknown';
        END_CASE;

        PollingActive := FALSE;
    ELSIF FB_ReadPowerLinkDiagnostics.Error THEN
        RetryCounter := RetryCounter + 1;
        IF RetryCounter >= MaxRetries THEN
            CommStatus := FALSE;
            ErrorCode := 101; // Custom error code for retry failure
            NodeHealth := 'No Response';
            PollingActive := FALSE;
        END_IF
    ELSE
        // Start timeout timer if no response
        ResponseTimeout(IN := TRUE);
        IF ResponseTimeout.Q THEN
            CommStatus := FALSE;
            ErrorCode := 102; // Timeout
            NodeHealth := 'Timeout';
            PollingActive := FALSE;
            ResponseTimeout(IN := FALSE);
        END_IF
    END_IF
END_IF
