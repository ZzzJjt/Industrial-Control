FUNCTION_BLOCK FB_PowerLink_Diagnostics
VAR_INPUT
    PollTrigger     : BOOL;        // Rising edge triggers new diagnostic read
    NodeID          : INT;         // Target control node ID
END_VAR

VAR_OUTPUT
    CommStatus      : BOOL;        // Communication active
    NodeHealthy     : BOOL;        // Node health status
    ErrorCode       : INT;         // Specific error code from node
    FaultDetected   : BOOL;        // TRUE if any fault is found
    LastUpdateOK    : BOOL;        // TRUE if last cycle was valid
END_VAR

VAR
    prevTrigger     : BOOL;
    readSuccess     : BOOL;
    rawDiagData     : WORD;        // Raw diagnostic word
    retryCount      : INT := 0;
    maxRetries      : INT := 3;
END_VAR

// Detect rising edge of PollTrigger
IF PollTrigger AND NOT prevTrigger THEN
    retryCount := 0;
    readSuccess := FALSE;

    // Simulate call to Managing Node function (replace with actual interface call)
    WHILE (retryCount < maxRetries) AND NOT readSuccess DO
        rawDiagData := ReadPowerLinkDiagnostics(NodeID); // Placeholder function
        readSuccess := ValidateData(rawDiagData);
        retryCount := retryCount + 1;
    END_WHILE

    IF readSuccess THEN
        CommStatus := (rawDiagData AND 16#0001) <> 0;
        NodeHealthy := (rawDiagData AND 16#0002) <> 0;
        ErrorCode := INT_TO_WORD((rawDiagData AND 16#FFF0) >> 4);
        FaultDetected := NOT NodeHealthy OR NOT CommStatus;
        LastUpdateOK := TRUE;
    ELSE
        CommStatus := FALSE;
        NodeHealthy := FALSE;
        ErrorCode := -1;
        FaultDetected := TRUE;
        LastUpdateOK := FALSE;
    END_IF
END_IF

prevTrigger := PollTrigger;

// Dummy validation logic
FUNCTION ValidateData : BOOL
VAR_INPUT
    diagWord : WORD;
END_VAR
ValidateData := (diagWord <> 16#FFFF);

// Placeholder interface simulation
FUNCTION ReadPowerLinkDiagnostics : WORD
VAR_INPUT
    node : INT;
END_VAR
// Simulate valid return
ReadPowerLinkDiagnostics := 16#1A3B;
