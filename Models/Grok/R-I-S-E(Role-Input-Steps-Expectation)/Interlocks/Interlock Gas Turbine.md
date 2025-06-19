FUNCTION_BLOCK TURBINE_INTERLOCK
VAR_INPUT
    Execute: BOOL;             // Cyclic execution trigger
    TT101: REAL;               // Exhaust temperature (°C), 0–800
    ST101: REAL;               // Turbine speed (RPM), 0–4000
    PT101: REAL;               // Compressor pressure (bar), 0–40
    PT102: REAL;               // Lube oil pressure (bar), 0–10
    VT101: REAL;               // Vibration (mm/s RMS), 0–20
    FD101: BOOL;               // Flame detector, TRUE: Flame present
    PT103: REAL;               // Fuel gas pressure (bar), 0–10
    FT101: REAL;               // Cooling water flow (L/min), 0–100
    SD101: BOOL;               // Surge detector, TRUE: Surge detected
    ES101: BOOL;               // Emergency stop, TRUE: Activated
END_VAR

VAR_OUTPUT
    Done: BOOL;                // TRUE when interlocks processed
    Error: BOOL;               // TRUE if error occurs
    ErrorID: DWORD;            // 0: No error, 1: Sensor fault
    FV101_Command: BOOL;       // Fuel Valve, FALSE: Close, TRUE: Open
    PRV101_Command: BOOL;      // Pressure Relief Valve, TRUE: Open
    ASV101_Command: BOOL;      // Anti-Surge Valve, TRUE: Open
    SC101_Command: REAL;       // Speed Controller, 0–100%
    MOT101_Command: BOOL;      // Mechanical Overspeed Trip, TRUE: Engage
    TA101_Command: BOOL;       // Alarm, TRUE: Active
    ESD101_Command: BOOL;      // Emergency Shutdown, TRUE: Active
    AuditMessage: STRING[80];  // Event or error log
END_VAR

VAR
    ExecuteEdge: BOOL;         // Edge detection for Execute
    SensorValid: BOOL;         // TRUE if sensors are valid
    InterlockTriggered: BOOL;  // TRUE if any interlock is active
END_VAR

// Reset outputs when not executing
IF NOT Execute THEN
    Done := FALSE;
    Error := FALSE;
    ErrorID := 0;
    FV101_Command := TRUE;     // Default: Open
    PRV101_Command := FALSE;   // Default: Closed
    ASV101_Command := FALSE;   // Default: Closed
    SC101_Command := 100.0;    // Default: Full speed
    MOT101_Command := FALSE;   // Default: Disengaged
    TA101_Command := FALSE;    // Default: Off
    ESD101_Command := FALSE;   // Default: Off
    AuditMessage := '';
    ExecuteEdge := FALSE;
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
        FV101_Command := TRUE;
        PRV101_Command := FALSE;
        ASV101_Command := FALSE;
        SC101_Command := 100.0;
        MOT101_Command := FALSE;
        TA101_Command := FALSE;
        ESD101_Command := FALSE;
        AuditMessage := 'Interlock monitoring started';
    END_IF;
    
    // Validate sensor readings
    SensorValid := TRUE;
    InterlockTriggered := FALSE;
    
    IF TT101 < 0.0 OR TT101 > 800.0 OR
       ST101 < 0.0 OR ST101 > 4000.0 OR
       PT101 < 0.0 OR PT101 > 40.0 OR
       PT102 < 0.0 OR PT102 > 10.0 OR
       VT101 < 0.0 OR VT101 > 20.0 OR
       FT101 < 0.0 OR FT101 > 100.0 THEN
        SensorValid := FALSE;
        Error := TRUE;
        ErrorID := 1; // Sensor fault
        AuditMessage := 'Sensor fault detected';
        FV101_Command := FALSE; // Close fuel valve
        PRV101_Command := TRUE; // Open relief valve
        ASV101_Command := TRUE; // Open anti-surge valve
        SC101_Command := 0.0;   // Stop turbine
        MOT101_Command := TRUE; // Engage overspeed trip
        TA101_Command := TRUE;  // Trigger alarm
        ESD101_Command := TRUE; // Initiate shutdown
        InterlockTriggered := TRUE;
        Done := TRUE;
        RETURN;
    END_IF;
    
    // Interlock logic
    IF SensorValid THEN
        // Overtemperature (> 650°C)
        IF TT101 > 650.0 THEN
            FV101_Command := FALSE;
            TA101_Command := TRUE;
            ESD101_Command := TRUE;
            InterlockTriggered := TRUE;
            AuditMessage := 'Overtemperature: ' + REAL_TO_STRING(TT101) + '°C';
        END_IF;
        
        // Overspeed (> 3600 RPM)
        IF ST101 > 3600.0 THEN
            FV101_Command := FALSE;
            MOT101_Command := TRUE;
            TA101_Command := TRUE;
            ESD101_Command := TRUE;
            InterlockTriggered := TRUE;
            AuditMessage := 'Overspeed: ' + REAL_TO_STRING(ST101) + ' RPM';
        END_IF;
        
        // Overpressure (> 30 bar)
        IF PT101 > 30.0 THEN
            PRV101_Command := TRUE;
            FV101_Command := FALSE;
            TA101_Command := TRUE;
            InterlockTriggered := TRUE;
            AuditMessage := 'Overpressure: ' + REAL_TO_STRING(PT101) + ' bar';
        END_IF;
        
        // Low Lubrication Pressure (< 1.5 bar)
        IF PT102 < 1.5 THEN
            FV101_Command := FALSE;
            TA101_Command := TRUE;
            ESD101_Command := TRUE;
            InterlockTriggered := TRUE;
            AuditMessage := 'Low lube pressure: ' + REAL_TO_STRING(PT102) + ' bar';
        END_IF;
        
        // High Vibration (> 10 mm/s)
        IF VT101 > 10.0 THEN
            SC101_Command := 50.0; // Reduce speed
            TA101_Command := TRUE;
            IF VT101 > 12.0 THEN   // Critical threshold
                ESD101_Command := TRUE;
            END_IF;
            InterlockTriggered := TRUE;
            AuditMessage := 'High vibration: ' + REAL_TO_STRING(VT101) + ' mm/s';
        END_IF;
        
        // Flame Failure
        IF NOT FD101 THEN
            FV101_Command := FALSE;
            TA101_Command := TRUE;
            ESD101_Command := TRUE;
            InterlockTriggered := TRUE;
            AuditMessage := 'Flame failure detected';
        END_IF;
        
        // Low Fuel Gas Pressure (< 5 bar)
        IF PT103 < 5.0 THEN
            FV101_Command := FALSE;
            TA101_Command := TRUE;
            ESD101_Command := TRUE;
            InterlockTriggered := TRUE;
            AuditMessage := 'Low fuel pressure: ' + REAL_TO_STRING(PT103) + ' bar';
        END_IF;
        
        // Low Cooling Water Flow (< 50 L/min)
        IF FT101 < 50.0 THEN
            SC101_Command := 50.0; // Reduce load
            TA101_Command := TRUE;
            IF FT101 < 40.0 THEN   // Critical threshold
                ESD101_Command := TRUE;
            END_IF;
            InterlockTriggered := TRUE;
            AuditMessage := 'Low cooling flow: ' + REAL_TO_STRING(FT101) + ' L/min';
        END_IF;
        
        // Compressor Surge
        IF SD101 THEN
            ASV101_Command := TRUE;
            SC101_Command := 50.0;
            TA101_Command := TRUE;
            InterlockTriggered := TRUE;
            AuditMessage := 'Compressor surge detected';
        END_IF;
        
        // Emergency Stop
        IF ES101 THEN
            FV101_Command := FALSE;
            TA101_Command := TRUE;
            ESD101_Command := TRUE;
            InterlockTriggered := TRUE;
            AuditMessage := 'Emergency stop activated';
        END_IF;
        
        // Reset commands if no interlocks triggered
        IF NOT InterlockTriggered THEN
            FV101_Command := TRUE;
            PRV101_Command := FALSE;
            ASV101_Command := FALSE;
            SC101_Command := 100.0;
            MOT101_Command := FALSE;
            TA101_Command := FALSE;
            ESD101_Command := FALSE;
            AuditMessage := 'Normal operation';
        END_IF;
        
        Done := TRUE;
    END_IF;
END_IF;
END_FUNCTION_BLOCK
