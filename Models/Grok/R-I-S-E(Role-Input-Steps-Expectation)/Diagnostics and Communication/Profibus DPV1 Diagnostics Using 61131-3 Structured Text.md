FUNCTION_BLOCK PROFIBUS_DPV1_DIAG
VAR_INPUT
    Execute: BOOL;             // Trigger diagnostic read on rising edge
    SlaveAddress: BYTE;        // Profibus DPV1 slave address (1–126)
    Timeout: TIME;             // Max wait time for response (e.g., T#1s)
END_VAR

VAR_OUTPUT
    Done: BOOL;                // TRUE when diagnostics retrieved
    Busy: BOOL;                // TRUE during operation
    Error: BOOL;               // TRUE if error occurs
    ErrorID: DWORD;            // 0: No error, 1: Invalid input, 2: Timeout, 3: Invalid response, 4: Comm loss
    Diagnostics: ARRAY[1..10] OF DIAG_DATA; // Diagnostic data
    DiagCount: INT;            // Number of valid diagnostic entries
    AuditMessage: STRING[80];  // Event or error log
END_VAR

VAR
    // Diagnostic data structure
    TYPE DIAG_DATA:
        STRUCT
            TypeID: BYTE;       // 1: CommError, 2: DeviceStatus, ..., 10: ManufacturerSpecific
            Code: INT;          // Error code (0: none, specific to type)
            Severity: BYTE;     // 0: Info, 1: Warning, 2: Critical
            Description: STRING[32]; // Human-readable message
        END_STRUCT;
    END_TYPE
    
    State: INT;                // State machine: 0=Idle, 1=Requesting, 2=Waiting, 3=Parsing, 4=Complete
    ExecuteEdge: BOOL;         // Edge detection for Execute
    ResponseTimer: TON;        // Timeout timer
    TimerStarted: BOOL;        // Tracks if timer is running
    ResponseValid: BOOL;       // Simulated response validity
    SimulatedResponse: ARRAY[1..10] OF INT; // Simulated diagnostic data
    i: INT;                    // Loop variable
END_VAR

// Reset outputs when not executing
IF NOT Execute THEN
    Done := FALSE;
    Busy := FALSE;
    Error := FALSE;
    ErrorID := 0;
    DiagCount := 0;
    FOR i := 1 TO 10 DO
        Diagnostics[i].TypeID := 0;
        Diagnostics[i].Code := 0;
        Diagnostics[i].Severity := 0;
        Diagnostics[i].Description := '';
    END_FOR;
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
        DiagCount := 0;
        FOR i := 1 TO 10 DO
            Diagnostics[i].TypeID := 0;
            Diagnostics[i].Code := 0;
            Diagnostics[i].Severity := 0;
            Diagnostics[i].Description := '';
        END_FOR;
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
            // Simulate sending DPV1 diagnostic request (placeholder for Profibus master API)
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
                // Simulate 1–10 diagnostic types
                DiagCount := RANDOM(1, 10);
                FOR i := 1 TO DiagCount DO
                    SimulatedResponse[i] := RANDOM(0, 100); // Simulate diagnostic data
                END_FOR;
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
                // Real system: Parse DPV1 diagnostic telegram (standard + extended diagnostics)
                FOR i := 1 TO DiagCount DO
                    Diagnostics[i].TypeID := i; // Types 1–10
                    CASE i OF
                        1: // CommError
                            Diagnostics[i].Code := SimulatedResponse[i] >= 80 ? 101 : 0;
                            Diagnostics[i].Severity := SimulatedResponse[i] >= 80 ? 2 : 0;
                            Diagnostics[i].Description := SimulatedResponse[i] >= 80 ? 
                                'Communication Error' : 'No Comm Issue';
                        2: // DeviceStatus
                            Diagnostics[i].Code := SimulatedResponse[i] >= 80 ? 102 : 0;
                            Diagnostics[i].Severity := SimulatedResponse[i] >= 80 ? 1 : 0;
                            Diagnostics[i].Description := SimulatedResponse[i] >= 80 ? 
                                'Device Fault' : 'Device OK';
                        3: // ParameterFault
                            Diagnostics[i].Code := SimulatedResponse[i] >= 80 ? 103 : 0;
                            Diagnostics[i].Severity := SimulatedResponse[i] >= 80 ? 2 : 0;
                            Diagnostics[i].Description := SimulatedResponse[i] >= 80 ? 
                                'Parameter Fault' : 'Parameters OK';
                        4: // HardwareWarning
                            Diagnostics[i].Code := SimulatedResponse[i] >= 80 ? 104 : 0;
                            Diagnostics[i].Severity := SimulatedResponse[i] >= 80 ? 1 : 0;
                            Diagnostics[i].Description := SimulatedResponse[i] >= 80 ? 
                                'Hardware Warning' : 'Hardware OK';
                        5: // OverrunError
                            Diagnostics[i].Code := SimulatedResponse[i] >= 80 ? 105 : 0;
                            Diagnostics[i].Severity := SimulatedResponse[i] >= 80 ? 2 : 0;
                            Diagnostics[i].Description := SimulatedResponse[i] >= 80 ? 
                                'Data Overrun' : 'No Overrun';
                        6: // OverflowError
                            Diagnostics[i].Code := SimulatedResponse[i] >= 80 ? 106 : 0;
                            Diagnostics[i].Severity := SimulatedResponse[i] >= 80 ? 2 : 0;
                            Diagnostics[i].Description := SimulatedResponse[i] >= 80 ? 
                                'Buffer Overflow' : 'No Overflow';
                        7: // SensorFault
                            Diagnostics[i].Code := SimulatedResponse[i] >= 80 ? 107 : 0;
                            Diagnostics[i].Severity := SimulatedResponse[i] >= 80 ? 2 : 0;
                            Diagnostics[i].Description := SimulatedResponse[i] >= 80 ? 
                                'Sensor Fault' : 'Sensor OK';
                        8: // ActuatorFault
                            Diagnostics[i].Code := SimulatedResponse[i] >= 80 ? 108 : 0;
                            Diagnostics[i].Severity := SimulatedResponse[i] >= 80 ? 2 : 0;
                            Diagnostics[i].Description := SimulatedResponse[i] >= 80 ? 
                                'Actuator Fault' : 'Actuator OK';
                        9: // FirmwareError
                            Diagnostics[i].Code := SimulatedResponse[i] >= 80 ? 109 : 0;
                            Diagnostics[i].Severity := SimulatedResponse[i] >= 80 ? 2 : 0;
                            Diagnostics[i].Description := SimulatedResponse[i] >= 80 ? 
                                'Firmware Error' : 'Firmware OK';
                        10: // ManufacturerSpecific
                            Diagnostics[i].Code := SimulatedResponse[i] >= 80 ? 110 : 0;
                            Diagnostics[i].Severity := SimulatedResponse[i] >= 80 ? 1 : 0;
                            Diagnostics[i].Description := SimulatedResponse[i] >= 80 ? 
                                'Vendor Error' : 'Vendor OK';
                    END_CASE;
                END_FOR;
                
                Done := TRUE;
                AuditMessage := 'Diagnostics parsed for Slave ' + BYTE_TO_STRING(SlaveAddress) + 
                                ', Count=' + INT_TO_STRING(DiagCount);
                IF DiagCount > 0 THEN
                    FOR i := 1 TO DiagCount DO
                        IF Diagnostics[i].Severity > 0 THEN
                            Error := TRUE;
                            ErrorID := 3; // Device error
                            AuditMessage := 'Slave ' + BYTE_TO_STRING(SlaveAddress) + 
                                            ': ' + Diagnostics[i].Description;
                            EXIT;
                        END_IF;
                    END_FOR;
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
