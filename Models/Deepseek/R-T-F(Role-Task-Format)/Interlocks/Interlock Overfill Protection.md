PROGRAM OverfillProtection
VAR_INPUT
    LevelSensor : REAL;       // Liquid level sensor value (e.g., meters)
    SensorFault : BOOL;       // Flag indicating sensor fault (TRUE = Faulty)
    ValveMalfunction : BOOL;  // Flag indicating valve malfunction (TRUE = Malfunctioning)
END_VAR

VAR_OUTPUT
    INLET_VALVE : BOOL;      // Inlet valve status (TRUE = Open, FALSE = Closed)
    SHUTDOWN : BOOL;         // Shutdown flag indicating an overfill condition
END_VAR

VAR
    HIGH_LEVEL_SETPOINT : REAL := 5.0;  // High-level setpoint in meters
    RESET_THRESHOLD : REAL := 4.0;      // Reset threshold in meters
    VALVE_CLOSED_LATCH : BOOL;          // Latch to keep valve closed after overfill
    FAULT_DETECTED : BOOL;              // Latch for sensor or valve fault
END_VAR

METHOD Execute : BOOL
BEGIN
    // Initialize outputs if not already initialized
    IF NOT VALVE_CLOSED_LATCH AND NOT FAULT_DETECTED THEN
        INLET_VALVE := TRUE; // Inlet valve initially open
        SHUTDOWN := FALSE;
    END_IF;

    // Check for sensor fault
    IF SensorFault THEN
        INLET_VALVE := FALSE; // Close inlet valve on sensor fault
        SHUTDOWN := TRUE;
        FAULT_DETECTED := TRUE; // Latch the fault condition
    END_IF;

    // Check for valve malfunction
    IF ValveMalfunction THEN
        INLET_VALVE := FALSE; // Close inlet valve on valve malfunction
        SHUTDOWN := TRUE;
        FAULT_DETECTED := TRUE; // Latch the fault condition
    END_IF;

    // Check for overfill condition
    IF LevelSensor > HIGH_LEVEL_SETPOINT AND NOT VALVE_CLOSED_LATCH AND NOT FAULT_DETECTED THEN
        INLET_VALVE := FALSE; // Close inlet valve on overfill
        SHUTDOWN := TRUE;
        VALVE_CLOSED_LATCH := TRUE; // Latch the valve closed condition
    END_IF;

    // Allow reset only if no faults and level drops below reset threshold
    IF LevelSensor < RESET_THRESHOLD AND NOT SensorFault AND NOT ValveMalfunction THEN
        VALVE_CLOSED_LATCH := FALSE; // Clear the valve closed latch
        FAULT_DETECTED := FALSE;     // Clear the fault latch
        INLET_VALVE := TRUE;         // Reopen inlet valve
        SHUTDOWN := FALSE;
    END_IF;

    RETURN TRUE;
END_METHOD

END_PROGRAM
