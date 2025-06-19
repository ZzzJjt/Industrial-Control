FUNCTION_BLOCK PROFIBUS_DIAG
VAR_INPUT
    Execute: BOOL;             // Trigger diagnostic read on rising edge
    SlaveAddress: BYTE;        // Profibus DP slave address (1–126)
    Timeout: TIME;             // Max wait time for response (e.g., T#1s)
END_VAR

VAR_OUTPUT
    Done: BOOL;                // TRUE when diagnostics retrieved successfully
    Busy: BOOL;                // TRUE during operation
    Error: BOOL;               // TRUE if error occurs
    ErrorID: DWORD;            // 0: No error, 1: Invalid address, 2: Timeout, 3: Invalid response, 4: Comm loss
    DeviceStatus: BYTE;        // 0: OK, 1: Warning, 2: Fault
    ErrorCode: INT;            // Slave-specific error code (0: no error)
    CommState: BYTE;           // 0: OK, 1: Degraded, 2: Failed
    AuditMessage: STRING[80];  // Event or error log
END_VAR

VAR
    State: INT;                // State machine: 0=Idle, 1=Requesting, 2=Waiting, 3=Parsing, 4=Complete
    ExecuteEdge: BOOL;         // Edge detection for Execute
    ResponseTimer: TON;        // Timeout timer
    TimerStarted: BOOL;        // Tracks if timer is running
    ResponseValid: BOOL;       // Simulated response validity
    SimulatedResponse: INT;    // Simulated diagnostic data
END_VAR

// Reset outputs when not executing
IF NOT Execute THEN
    Done := FALSE;
    Busy := FALSE;
    Error := FALSE;
    ErrorID := 0;
    DeviceStatus := 0;
    ErrorCode := 0;
    CommState := 0;
    AuditMessage := '';
    State := 0;
    ExecuteEdge := FALSE;
    ResponseTimer(IN := FALSE);
    TimerStarted := FALSE;
    RETURN;
END_IF;

// Cyclic execution
IF Execute THEN
    // Initialize on rising edge
    IF NOT ExecuteEdge THEN
        ExecuteEdge := TRUE;
        State := 1; // Start requesting diagnostics
        Done := FALSE;
        Busy := TRUE;
        Error := FALSE;
        ErrorID := 0;
        DeviceStatus := 0;
        ErrorCode := 0;
        CommState := 0;
        AuditMessage := 'Requesting diagnostics for Slave ' + BYTE_TO_STRING(SlaveAddress);
        
        // Validate inputs
        IF SlaveAddress < 1 OR SlaveAddress > 126 THEN
            Error := TRUE;
            ErrorID := 1; // Invalid address
            AuditMessage := 'Invalid SlaveAddress: ' + BYTE_TO_STRING(SlaveAddress);
            State := 0;
            Busy := FALSE;
            RETURN;
        END_IF;
        
        IF Timeout < T#0ms THEN
            Error := TRUE;
            ErrorID := 1; // Invalid timeout
            AuditMessage := 'Invalid Timeout';
            State := 0;
            Busy := FALSE;
            RETURN;
        END_IF;
    END_IF;
    
    // State machine
    CASE State OF
        0: // Idle
            Done := TRUE;
            Busy := FALSE;
            AuditMessage := 'Operation completed';
        
        1: // Requesting
            // Simulate sending diagnostic request (placeholder for Profibus DP master API)
            // Real system: Use DP master function (e.g., Siemens SFC14 "DPRD_DAT" for diagnostics)
            ResponseTimer(IN := TRUE, PT := Timeout);
            TimerStarted := TRUE;
            State := 2; // Move to Waiting
            AuditMessage := 'Waiting for response from Slave ' + BYTE_TO_STRING(SlaveAddress);
        
        2: // Waiting
            IF ResponseTimer.Q THEN
                Error := TRUE;
                ErrorID := 2; // Timeout
                AuditMessage := 'Timeout for Slave ' + BYTE_TO_STRING(SlaveAddress);
                State := 4; // Complete with error
                ResponseTimer(IN := FALSE);
                TimerStarted := FALSE;
                RETURN;
            END_IF;
            
            // Simulate response arrival (placeholder)
            IF RANDOM(0, 100) < 90 THEN // 90% chance of response
                ResponseValid := TRUE;
                SimulatedResponse := RANDOM(0, 100); // Simulate diagnostic data
                State := 3; // Move to Parsing
                ResponseTimer(IN := FALSE);
                TimerStarted := FALSE;
            END_IF;
            
            // Simulate communication loss (1% chance)
            IF RANDOM(0, 1000) > 990 THEN
                Error := TRUE;
                ErrorID := 4; // Communication loss
                AuditMessage := 'Communication loss for Slave ' + BYTE_TO_STRING(SlaveAddress);
                State := 4;
                ResponseTimer(IN := FALSE);
                TimerStarted := FALSE;
            END_IF;
        
        3: // Parsing
            IF ResponseValid THEN
                // Parse simulated response
                // Real system: Parse Profibus DP diagnostic telegram (e.g., 6-byte standard + extended diagnostics)
                CommState := 0; // OK
                DeviceStatus := 0; // OK
                ErrorCode := 0; // No error
                
                IF SimulatedResponse >= 80 AND SimulatedResponse < 90 THEN
                    CommState := 1; // Degraded
                    DeviceStatus := 1; // Warning
                    ErrorCode := 101; // Simulated minor error
                ELSIF SimulatedResponse >= 90 THEN
                    CommState := 2; // Failed
                    DeviceStatus := 2; // Fault
                    ErrorCode := 102; // Simulated critical error
                END_IF;
                
                Done := TRUE;
                AuditMessage := 'Diagnostics OK for Slave ' + BYTE_TO_STRING(SlaveAddress) + 
                                ', Status=' + BYTE_TO_STRING(DeviceStatus);
                IF DeviceStatus > 0 THEN
                    Error := TRUE;
                    ErrorID := 3; // Device error
                    AuditMessage := 'Slave ' + BYTE_TO_STRING(SlaveAddress) + 
                                    ': Error Code ' + INT_TO_STRING(ErrorCode);
                END_IF;
                State := 4;
            ELSE
                Error := TRUE;
                ErrorID := 3; // Invalid response
                AuditMessage := 'Invalid response from Slave ' + BYTE_TO_STRING(SlaveAddress);
                State := 4;
            END_IF;
        
        4: // Complete
            Done := TRUE;
            Busy := FALSE;
            IF Error THEN
                AuditMessage := 'Completed with error for Slave ' + BYTE_TO_STRING(SlaveAddress);
            ELSE
                AuditMessage := 'Diagnostics completed successfully';
            END_IF;
            State := 0;
    END_CASE;
END_IF;
END_FUNCTION_BLOCK
