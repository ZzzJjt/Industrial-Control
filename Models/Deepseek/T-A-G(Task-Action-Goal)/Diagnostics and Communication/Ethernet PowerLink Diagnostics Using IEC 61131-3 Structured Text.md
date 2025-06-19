FUNCTION_BLOCK POWERLINK_DIAGNOSTICS
VAR_INPUT
    ENABLE : BOOL; // Enable diagnostics collection
END_VAR

VAR_OUTPUT
    COMM_STATUS : ARRAY[1..10] OF BOOL; // Communication status of each node
    ERROR_CODES : ARRAY[1..10] OF INT;   // Error codes of each node
    NODE_HEALTH : ARRAY[1..10] OF INT;   // Health status of each node
    NUM_NODES : INT := 10;              // Number of nodes
    STATUS_MESSAGE : STRING[100];        // Status message
    ALARM : BOOL;                       // Alarm signal if any issues detected
END_VAR

VAR
    last_comm_status : ARRAY[1..10] OF BOOL;
    last_error_codes : ARRAY[1..10] OF INT;
    last_node_health : ARRAY[1..10] OF INT;
    timerDelay : TON;                   // On-Delay Timer
    timerSetTime : TIME := T#5s;        // 5-second delay
    currentNodeIndex : INT := 1;        // Current node index for cyclic processing
    refreshCycleComplete : BOOL := FALSE;// Flag to indicate cycle completion
    errorCode : INT := 0;               // Error code for fault handling
    errorMessage : STRING[100];         // Error message for fault handling
END_VAR

METHOD ReadDiagnosticData : BOOL
VAR_INPUT
    nodeIndex : INT;
END_VAR
VAR
    success : BOOL := TRUE;
BEGIN
    // Simulate reading diagnostic data from the managing node (MN)
    // Replace with actual API calls in real application
    IF nodeIndex < 1 OR nodeIndex > NUM_NODES THEN
        success := FALSE;
        errorCode := 1;
        errorMessage := 'Invalid node index.';
    ELSE
        // Simulate reading communication status
        COMM_STATUS[nodeIndex] := TRUE; // Example: Assume communication is always true for demonstration
        // Simulate reading error codes
        ERROR_CODES[nodeIndex] := 0;    // Example: Assume no errors for demonstration
        // Simulate reading node health
        NODE_HEALTH[nodeIndex] := 100;  // Example: Assume full health for demonstration
    END_IF;

    RETURN success;
END_METHOD

METHOD ProcessDiagnosticData : BOOL
VAR_INPUT
    nodeIndex : INT;
END_VAR
VAR
    issueDetected : BOOL := FALSE;
BEGIN
    // Check for changes in communication status
    IF COMM_STATUS[nodeIndex] <> last_comm_status[nodeIndex] THEN
        last_comm_status[nodeIndex] := COMM_STATUS[nodeIndex];
        IF NOT COMM_STATUS[nodeIndex] THEN
            issueDetected := TRUE;
            STATUS_MESSAGE := CONCAT('Node ', TO_STRING(nodeIndex), ': Communication lost.');
        END_IF;
    END_IF;

    // Check for non-zero error codes
    IF ERROR_CODES[nodeIndex] <> 0 THEN
        issueDetected := TRUE;
        STATUS_MESSAGE := CONCAT('Node ', TO_STRING(nodeIndex), ': Error Code ', TO_STRING(ERROR_CODES[nodeIndex]), '.');
    END_IF;

    // Check for reduced node health
    IF NODE_HEALTH[nodeIndex] < 80 THEN
        issueDetected := TRUE;
        STATUS_MESSAGE := CONCAT('Node ', TO_STRING(nodeIndex), ': Health below threshold (', TO_STRING(NODE_HEALTH[nodeIndex]), ').');
    END_IF;

    RETURN issueDetected;
END_METHOD

METHOD Execute : BOOL
BEGIN
    IF NOT ENABLE THEN
        ALARM := FALSE;
        RETURN FALSE;
    END_IF;

    // Start the timer for cyclic data collection
    timerDelay(IN := TRUE, PT := timerSetTime);
    IF timerDelay.Q THEN
        timerDelay(IN := FALSE); // Reset the timer after it expires

        // Cyclically read and process diagnostic data for each node
        IF currentNodeIndex <= NUM_NODES THEN
            IF NOT ReadDiagnosticData(currentNodeIndex) THEN
                // Handle error
                DBG(errorMessage);
            ELSE
                IF ProcessDiagnosticData(currentNodeIndex) THEN
                    ALARM := TRUE;
                END_IF;
            END_IF;
            currentNodeIndex := currentNodeIndex + 1;
        ELSE
            currentNodeIndex := 1; // Reset to start of cycle
            refreshCycleComplete := TRUE;
        END_IF;
    END_IF;

    IF refreshCycleComplete THEN
        refreshCycleComplete := FALSE;
    END_IF;

    // Delay to prevent busy-waiting
    SLEEP(T#100ms);

    RETURN TRUE;
END_METHOD

END_FUNCTION_BLOCK



