FUNCTION_BLOCK PRESSURE_RELIEF_INTERLOCK
VAR_INPUT
    Execute: BOOL;             // Cyclic execution trigger
    PT101: REAL;               // Pressure (bar), 0–20
    SensorFault: BOOL;         // TRUE if sensor failure detected
    ValveFault: BOOL;          // TRUE if valve malfunction detected
    Reset: BOOL;               // Manual reset to clear shutdown
END_VAR

VAR_OUTPUT
    Done: BOOL;                // TRUE when interlocks processed
    Error: BOOL;               // TRUE if error occurs
    ErrorID: DWORD;            // 0: No error, 1: Sensor fault, 2: Valve fault
    RELIEF_VALVE: BOOL;        // Relief Valve, TRUE: Open, FALSE: Closed
    SHUTDOWN_LATCH: BOOL;      // TRUE: System in shutdown state
    Alarm: BOOL;               // TRUE if interlock or fault triggered
    AuditMessage: STRING[80];  // Event or error log
END_VAR

VAR
    // Constants
    HIGH_LIMIT: REAL := 15.0;  // Overpressure threshold (bar)
    RESET_LIMIT: REAL := 12.0; // Safe reset threshold (bar)
    
    ExecuteEdge: BOOL;         // Edge detection for Execute
    SensorValid: BOOL;         // TRUE if PT101 is valid
    InterlockTriggered: BOOL;  // TRUE if interlock is active
    ResetEdge: BOOL;           // Edge detection for Reset
END_VAR

// Reset outputs when not executing (fail-safe)
IF NOT Execute THEN
    Done := FALSE;
    Error := FALSE;
    ErrorID := 0;
    RELIEF_VALVE := TRUE;      // Default: Open (fail-safe)
    SHUTDOWN_LATCH := FALSE;
    Alarm := FALSE;
    AuditMessage := '';
    ExecuteEdge := FALSE;
    ResetEdge := FALSE;
    RETURN;
END_IF;

// Cyclic execution
IF Execute THEN
    // Initialize on rising edge
    IF NOT ExecuteEdge THEN
        ExecuteEdge := TRUE;
        Done := FALSE;
        Error := FALSE;
        ErrorID := 0;
        RELIEF_VALVE := FALSE; // Default: Closed (normal operation)
        SHUTDOWN_LATCH := FALSE;
        Alarm := FALSE;
        AuditMessage := 'Interlock monitoring started';
    END_IF;
    
    // Validate sensor reading
    SensorValid := TRUE;
    InterlockTriggered := FALSE;
    
    IF PT101 < 0.0 OR PT101 > 20.0 OR SensorFault THEN
        SensorValid := FALSE;
        Error := TRUE;
        ErrorID := 1; // Sensor fault
        AuditMessage := 'Sensor fault detected';
        RELIEF_VALVE := TRUE;  // Open relief valve
        SHUTDOWN_LATCH := TRUE;
        Alarm := TRUE;
        InterlockTriggered := TRUE;
        Done := TRUE;
    END_IF;
    
    // Check valve fault
    IF ValveFault THEN
        Error := TRUE;
        ErrorID := 2; // Valve fault
        AuditMessage := 'Valve malfunction detected';
        RELIEF_VALVE := TRUE;  // Open relief valve
        SHUTDOWN_LATCH := TRUE;
        Alarm := TRUE;
        InterlockTriggered := TRUE;
        Done := TRUE;
    END_IF;
    
    // Interlock logic for overpressure
    IF SensorValid AND NOT ValveFault THEN
        // High Pressure (> 15 bar)
        IF PT101 > HIGH_LIMIT THEN
            RELIEF_VALVE := TRUE;  // Open relief valve
            SHUTDOWN_LATCH := TRUE;
            Alarm := TRUE;
            InterlockTriggered := TRUE;
            AuditMessage := 'Overpressure: ' + REAL_TO_STRING(PT101) + ' bar';
        END_IF;
        
        // Maintain open until pressure < 12 bar
        IF SHUTDOWN_LATCH AND PT101 >= RESET_LIMIT THEN
            RELIEF_VALVE := TRUE;  // Keep valve open
            Alarm := TRUE;
            AuditMessage := 'Pressure above reset: ' + REAL_TO_STRING(PT101) + ' bar';
        END_IF;
    END_IF;
    
    // Shutdown latch and manual reset
    IF SHUTDOWN_LATCH THEN
        RELIEF_VALVE := TRUE;  // Ensure valve remains open
        SHUTDOWN_LATCH := TRUE;
        IF Reset AND NOT ResetEdge THEN
            ResetEdge := TRUE;
            IF PT101 < RESET_LIMIT AND SensorValid AND NOT ValveFault THEN
                SHUTDOWN_LATCH := FALSE;
                RELIEF_VALVE := FALSE; // Close valve
                Alarm := FALSE;
                AuditMessage := 'Manual reset successful';
            ELSE
                AuditMessage := 'Reset failed: Unsafe conditions';
            END_IF;
        END_IF;
    END_IF;
    
    // Clear reset edge
    IF NOT Reset THEN
        ResetEdge := FALSE;
    END_IF;
    
    // Set normal operation message if no interlocks
    IF NOT InterlockTriggered AND NOT SHUTDOWN_LATCH THEN
        RELIEF_VALVE := FALSE; // Close valve
        AuditMessage := 'Normal operation';
        Alarm := FALSE;
    END_IF;
    
    Done := TRUE;
END_IF;
END_FUNCTION_BLOCK
