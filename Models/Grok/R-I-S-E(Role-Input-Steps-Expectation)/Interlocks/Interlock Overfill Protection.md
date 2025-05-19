FUNCTION_BLOCK SUBSEA_WELLHEAD_INTERLOCK
VAR_INPUT
    Execute: BOOL;             // Cyclic execution trigger
    PT101: REAL;               // Pressure (psi), 0–2000
    TT101: REAL;               // Temperature (°C), 0–150
    FT101: REAL;               // Flow rate (m³/h), 0–500
    Reset: BOOL;               // Manual reset to clear shutdown
END_VAR

VAR_OUTPUT
    Done: BOOL;                // TRUE when interlocks processed
    Error: BOOL;               // TRUE if error occurs
    ErrorID: DWORD;            // 0: No error, 1: Sensor fault
    MV101_Command: BOOL;       // Master Valve, FALSE: Closed, TRUE: Open
    Shutdown: BOOL;            // TRUE: System in shutdown state
    Alarm: BOOL;               // TRUE if interlock triggered
    AuditMessage: STRING[80];  // Event or error log
END_VAR

VAR
    ExecuteEdge: BOOL;         // Edge detection for Execute
    SensorValid: BOOL;         // TRUE if sensors are valid
    InterlockTriggered: BOOL;  // TRUE if any interlock is active
    ShutdownLatch: BOOL;       // Latch for shutdown state
    ResetEdge: BOOL;           // Edge detection for Reset
END_VAR

// Reset outputs when not executing
IF NOT Execute THEN
    Done := FALSE;
    Error := FALSE;
    ErrorID := 0;
    MV101_Command := FALSE;    // Default: Closed (fail-safe)
    Shutdown := FALSE;
    Alarm := FALSE;
    AuditMessage := '';
    ExecuteEdge := FALSE;
    ShutdownLatch := FALSE;
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
        MV101_Command := TRUE; // Default: Open (normal operation)
        Shutdown := FALSE;
        Alarm := FALSE;
        AuditMessage := 'Interlock monitoring started';
        ShutdownLatch := FALSE;
    END_IF;
    
    // Validate sensor readings
    SensorValid := TRUE;
    InterlockTriggered := FALSE;
    
    IF PT101 < 0.0 OR PT101 > 2000.0 OR
       TT101 < 0.0 OR TT101 > 150.0 OR
       FT101 < 0.0 OR FT101 > 500.0 THEN
        SensorValid := FALSE;
        Error := TRUE;
        ErrorID := 1; // Sensor fault
        AuditMessage := 'Sensor fault detected';
        MV101_Command := FALSE; // Close master valve
        ShutdownLatch := TRUE;
        Shutdown := TRUE;
        Alarm := TRUE;
        InterlockTriggered := TRUE;
        Done := TRUE;
    END_IF;
    
    // Interlock logic
    IF SensorValid AND NOT ShutdownLatch THEN
        // High Pressure (> 1500 psi)
        IF PT101 > 1500.0 THEN
            MV101_Command := FALSE; // Close master valve
            ShutdownLatch := TRUE;
            Shutdown := TRUE;
            Alarm := TRUE;
            InterlockTriggered := TRUE;
            AuditMessage := 'High pressure: ' + REAL_TO_STRING(PT101) + ' psi';
        END_IF;
        
        // Low Flow (< 50 m³/h, possible leak)
        IF FT101 < 50.0 THEN
            MV101_Command := FALSE; // Close master valve
            ShutdownLatch := TRUE;
            Shutdown := TRUE;
            Alarm := TRUE;
            InterlockTriggered := TRUE;
            AuditMessage := 'Low flow: ' + REAL_TO_STRING(FT101) + ' m³/h';
        END_IF;
        
        // High Temperature (> 120°C)
        IF TT101 > 120.0 THEN
            MV101_Command := FALSE; // Close master valve
            ShutdownLatch := TRUE;
            Shutdown := TRUE;
            Alarm := TRUE;
            InterlockTriggered := TRUE;
            AuditMessage := 'High temperature: ' + REAL_TO_STRING(TT101) + '°C';
        END_IF;
    END_IF;
    
    // Shutdown latch and manual reset
    IF ShutdownLatch THEN
        MV101_Command := FALSE; // Keep valve closed
        Shutdown := TRUE;
        IF Reset AND NOT ResetEdge THEN
            ResetEdge := TRUE;
            IF PT101 <= 1500.0 AND FT101 >= 50.0 AND TT101 <= 120.0 THEN
                ShutdownLatch := FALSE;
                Shutdown := FALSE;
                MV101_Command := TRUE; // Reopen valve
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
    IF NOT InterlockTriggered AND NOT ShutdownLatch THEN
        AuditMessage := 'Normal operation';
        Alarm := FALSE;
    END_IF;
    
    Done := TRUE;
END_IF;
END_FUNCTION_BLOCK
