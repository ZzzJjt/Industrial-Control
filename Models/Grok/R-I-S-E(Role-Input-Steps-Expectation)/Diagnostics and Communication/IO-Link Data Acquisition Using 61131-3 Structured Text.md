FUNCTION_BLOCK IOLINK_READ_VALUES
VAR_INPUT
    EXECUTE: BOOL;             // Trigger read operation on rising edge
    MASTER_ID: BYTE;           // IO-Link master ID (1–255)
    CHANNEL: BYTE;             // IO-Link channel/port (1–8)
    RETRY_COUNT: INT := 3;     // Number of retry attempts (1–5)
END_VAR

VAR_OUTPUT
    DONE: BOOL;                // TRUE when all reads complete successfully
    ERROR: BOOL;               // TRUE if any read fails after retries
    ERROR_CODE: INT;           // 0: No error, 1: Invalid input, 2: Timeout, 3: CRC error, 4: Master fault
    VALUES: ARRAY[1..5] OF REAL; // Retrieved process values
    STATUSES: ARRAY[1..5] OF INT; // 0: Success, 1: Timeout, 2: CRC error, 3: Retry limit exceeded
    AUDIT_MESSAGE: STRING[80]; // Operation or error log
END_VAR

VAR
    State: INT;                // State machine: 0=Idle, 1–5=Reading value, 6=Complete
    ExecuteEdge: BOOL;         // Edge detection for EXECUTE
    RetryCounter: INT;         // Current retry attempt
    CurrentValue: INT;         // Index of value being read (1–5)
    SimulatedResponse: INT;    // Simulated read response (placeholder)
    ResponseValid: BOOL;       // Simulated response validity
    ReadTimer: TON;            // Timer for read timeout (500ms)
    TimerStarted: BOOL;        // Tracks if timer is running
END_VAR

// Reset outputs when not executing
IF NOT EXECUTE THEN
    DONE := FALSE;
    ERROR := FALSE;
    ERROR_CODE := 0;
    FOR i := 1 TO 5 DO
        VALUES[i] := 0.0;
        STATUSES[i] := 0;
    END_FOR;
    AUDIT_MESSAGE := '';
    State := 0;
    ExecuteEdge := FALSE;
    RetryCounter := 0;
    CurrentValue := 0;
    ReadTimer(IN := FALSE);
    TimerStarted := FALSE;
    RETURN;
END_IF;

// Cyclic execution
IF EXECUTE THEN
    // Initialize on rising edge
    IF NOT ExecuteEdge THEN
        ExecuteEdge := TRUE;
        State := 1; // Start reading first value
        RetryCounter := 0;
        CurrentValue := 1;
        DONE := FALSE;
        ERROR := FALSE;
        ERROR_CODE := 0;
        FOR i := 1 TO 5 DO
            VALUES[i] := 0.0;
            STATUSES[i] := 0;
        END_FOR;
        AUDIT_MESSAGE := 'Started reading for Master ' + BYTE_TO_STRING(MASTER_ID) + 
                         ', Channel ' + BYTE_TO_STRING(CHANNEL);
        
        // Validate inputs
        IF MASTER_ID < 1 OR MASTER_ID > 255 OR CHANNEL < 1 OR CHANNEL > 8 OR
           RETRY_COUNT < 1 OR RETRY_COUNT > 5 THEN
            ERROR := TRUE;
            ERROR_CODE := 1; // Invalid input
            AUDIT_MESSAGE := 'Invalid input parameters';
            State := 0;
            RETURN;
        END_IF;
    END_IF;
    
    // State machine for reading values
    CASE State OF
        0: // Idle
            DONE := TRUE;
            AUDIT_MESSAGE := 'Operation completed';
        
        1..5: // Reading value
            IF NOT TimerStarted THEN
                // Initiate read (placeholder for IO-Link API call)
                ReadTimer(IN := TRUE, PT := T#500ms); // 500ms timeout
                TimerStarted := TRUE;
                AUDIT_MESSAGE := 'Reading value ' + INT_TO_STRING(CurrentValue) + 
                                 ' for Master ' + BYTE_TO_STRING(MASTER_ID);
            END_IF;
            
            IF ReadTimer.Q THEN
                // Simulate IO-Link master response
                // In real system, replace with actual read call (e.g., ISDU read)
                SimulatedResponse := RANDOM(0, 100); // 0–79: success, 80–89: timeout, 90–99: CRC error
                ResponseValid := SimulatedResponse < 90; // 90% valid responses
                
                ReadTimer(IN := FALSE);
                TimerStarted := FALSE;
                
                IF ResponseValid THEN
                    IF SimulatedResponse < 80 THEN
                        // Success
                        VALUES[CurrentValue] := REAL(SimulatedResponse) * 0.1; // Example value
                        STATUSES[CurrentValue] := 0; // Success
                        AUDIT_MESSAGE := 'Value ' + INT_TO_STRING(CurrentValue) + 
                                         ': Success, Value=' + REAL_TO_STRING(VALUES[CurrentValue]);
                        CurrentValue := CurrentValue + 1;
                        RetryCounter := 0;
                        IF CurrentValue > 5 THEN
                            State := 6; // Complete
                        ELSE
                            State := CurrentValue;
                        END_IF;
                    ELSE
                        // Timeout
                        STATUSES[CurrentValue] := 1; // Timeout
                        AUDIT_MESSAGE := 'Value ' + INT_TO_STRING(CurrentValue) + 
                                         ': Timeout';
                        RetryCounter := RetryCounter + 1;
                        IF RetryCounter >= RETRY_COUNT THEN
                            ERROR := TRUE;
                            ERROR_CODE := 2; // Timeout after retries
                            STATUSES[CurrentValue] := 3; // Retry limit exceeded
                            State := 6; // Complete with error
                        END_IF;
                    END_IF;
                ELSE
                    // CRC error
                    STATUSES[CurrentValue] := 2; // CRC error
                    AUDIT_MESSAGE := 'Value ' + INT_TO_STRING(CurrentValue) + 
                                     ': CRC error';
                    RetryCounter := RetryCounter + 1;
                    IF RetryCounter >= RETRY_COUNT THEN
                        ERROR := TRUE;
                        ERROR_CODE := 3; // CRC error after retries
                        STATUSES[CurrentValue] := 3; // Retry limit exceeded
                        State := 6; // Complete with error
                    END_IF;
                END_IF;
            END_IF;
        
        6: // Complete
            DONE := TRUE;
            IF ERROR THEN
                AUDIT_MESSAGE := 'Completed with errors';
            ELSE
                AUDIT_MESSAGE := 'All values read successfully';
            END_IF;
            State := 0; // Return to idle
    END_CASE;
    
    // Simulate master fault (e.g., no response)
    IF RANDOM(0, 1000) > 995 THEN // 0.5% chance
        ERROR := TRUE;
        ERROR_CODE := 4; // Master fault
        AUDIT_MESSAGE := 'Master ' + BYTE_TO_STRING(MASTER_ID) + ': Fault';
        State := 0;
    END_IF;
END_IF;
END_FUNCTION_BLOCK
