FUNCTION_BLOCK POWERLINK_DIAG
VAR_INPUT
    EXECUTE: BOOL;             // Cyclic execution trigger
    NODE_ID: BYTE;             // PowerLink node ID (1–240)
    POLL_INTERVAL: TIME;       // Polling interval (e.g., T#1s)
END_VAR

VAR_OUTPUT
    DONE: BOOL;                // TRUE when diagnostics retrieved successfully
    ERROR: BOOL;               // TRUE if error occurs
    ERROR_CODE: INT;           // 0: No error, 1: Invalid NODE_ID, 2: Timeout, 3: Invalid response
    COMM_STATUS: BOOL;         // TRUE: Node communication active
    NODE_ERROR: INT;           // Node-specific error code (0: no error)
    HEALTH_STATUS: INT;        // 0: Healthy, 1: Warning, 2: Critical
    AUDIT_MESSAGE: STRING[80]; // Diagnostic event or error log
END_VAR

VAR
    PollTimer: TON;            // Timer for polling interval
    ExecuteEdge: BOOL;         // Edge detection for EXECUTE
    PrevCommStatus: BOOL;      // Previous communication status
    ResponseValid: BOOL;       // Simulated response validity
    SimulatedResponse: INT;    // Simulated diagnostic data
END_VAR

// Reset outputs when not executing
IF NOT EXECUTE THEN
    DONE := FALSE;
    ERROR := FALSE;
    ERROR_CODE := 0;
    COMM_STATUS := FALSE;
    NODE_ERROR := 0;
    HEALTH_STATUS := 0;
    AUDIT_MESSAGE := '';
    PollTimer(IN := FALSE);
    ExecuteEdge := FALSE;
    PrevCommStatus := FALSE;
    RETURN;
END_IF;

// Cyclic execution
IF EXECUTE THEN
    // Initialize on first cycle
    IF NOT ExecuteEdge THEN
        ExecuteEdge := TRUE;
        AUDIT_MESSAGE := 'Initialized for Node ' + BYTE_TO_STRING(NODE_ID);
        PollTimer(IN := FALSE, PT := POLL_INTERVAL);
    END_IF;
    
    // Reset outputs for this cycle
    DONE := FALSE;
    ERROR := FALSE;
    ERROR_CODE := 0;
    
    // Validate NODE_ID (1–240 per PowerLink spec)
    IF NODE_ID < 1 OR NODE_ID > 240 THEN
        ERROR := TRUE;
        ERROR_CODE := 1; // Invalid NODE_ID
        AUDIT_MESSAGE := 'Invalid NODE_ID: ' + BYTE_TO_STRING(NODE_ID);
        RETURN;
    END_IF;
    
    // Manage polling timer
    PollTimer(IN := TRUE, PT := POLL_INTERVAL);
    
    // Process diagnostics when timer elapses
    IF PollTimer.Q THEN
        // Simulate MN query (placeholder for actual PowerLink API call)
        // Assume response: COMM_STATUS, NODE_ERROR, HEALTH_STATUS
        // In real system, replace with CANopen SDO read or equivalent
        ResponseValid := TRUE; // Simulated valid response
        SimulatedResponse := RANDOM(0, 100); // Simulate node response
        
        // Parse response
        IF ResponseValid THEN
            // Simulate communication status
            COMM_STATUS := SimulatedResponse < 80; // 80% chance of connection
            NODE_ERROR := 0;
            HEALTH_STATUS := 0; // Healthy
            
            // Simulate errors
            IF SimulatedResponse >= 80 AND SimulatedResponse < 90 THEN
                NODE_ERROR := 101; // Simulated timeout
                HEALTH_STATUS := 1; // Warning
            ELSIF SimulatedResponse >= 90 THEN
                NODE_ERROR := 102; // Simulated critical error
                HEALTH_STATUS := 2; // Critical
            END_IF;
            
            // Detect status changes
            IF COMM_STATUS <> PrevCommStatus THEN
                IF NOT COMM_STATUS THEN
                    AUDIT_MESSAGE := 'Node ' + BYTE_TO_STRING(NODE_ID) + 
                                     ': Connection Lost, Error ' + INT_TO_STRING(NODE_ERROR);
                ELSE
                    AUDIT_MESSAGE := 'Node ' + BYTE_TO_STRING(NODE_ID) + 
                                     ': Connection Restored';
                END_IF;
            ELSIF COMM_STATUS THEN
                IF HEALTH_STATUS = 0 THEN
                    AUDIT_MESSAGE := 'Node ' + BYTE_TO_STRING(NODE_ID) + ': Healthy';
                ELSIF HEALTH_STATUS = 1 THEN
                    AUDIT_MESSAGE := 'Node ' + BYTE_TO_STRING(NODE_ID) + 
                                     ': Warning, Error ' + INT_TO_STRING(NODE_ERROR);
                ELSE
                    AUDIT_MESSAGE := 'Node ' + BYTE_TO_STRING(NODE_ID) + 
                                     ': Critical, Error ' + INT_TO_STRING(NODE_ERROR);
                END_IF;
            ELSE
                AUDIT_MESSAGE := 'Node ' + BYTE_TO_STRING(NODE_ID) + 
                                 ': Disconnected, Error ' + INT_TO_STRING(NODE_ERROR);
            END_IF;
            
            // Set outputs
            DONE := TRUE;
            IF NOT COMM_STATUS OR HEALTH_STATUS > 0 THEN
                ERROR := TRUE;
                ERROR_CODE := 2; // Timeout or node error
                IF HEALTH_STATUS = 2 THEN
                    ERROR_CODE := 3; // Critical error
                END_IF;
            END_IF;
        ELSE
            // Invalid response
            ERROR := TRUE;
            ERROR_CODE := 3; // Invalid response
            COMM_STATUS := FALSE;
            NODE_ERROR := 0;
            HEALTH_STATUS := 2; // Assume critical if response fails
            AUDIT_MESSAGE := 'Node ' + BYTE_TO_STRING(NODE_ID) + 
                             ': Invalid response from MN';
        END_IF;
        
        PrevCommStatus := COMM_STATUS;
        PollTimer(IN := FALSE); // Reset timer
    END_IF;
END_IF;
END_FUNCTION_BLOCK
