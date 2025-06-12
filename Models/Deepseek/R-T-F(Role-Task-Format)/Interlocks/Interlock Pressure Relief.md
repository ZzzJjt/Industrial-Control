PROGRAM PressureReliefInterlock
VAR_INPUT
    PT_101 : REAL;       // Pressure Transmitter Value (bar)
    SensorFault : BOOL;   // Flag indicating sensor fault (TRUE = Faulty)
    ValveFault : BOOL;    // Flag indicating valve fault (TRUE = Malfunctioning)
END_VAR

VAR_OUTPUT
    RELIEF_VALVE : BOOL;  // Relief Valve Status (TRUE = Open, FALSE = Closed)
    ALARM : BOOL;         // Alarm Flag indicating a fault condition
END_VAR

VAR
    HIGH_PRESSURE_LIMIT : REAL := 15.0;  // High-pressure setpoint in bar
    RESET_THRESHOLD : REAL := 12.0;      // Reset threshold in bar
    VALVE_OPEN_LATCH : BOOL;             // Latch to keep valve open after overpressure
    FAULT_DETECTED : BOOL;               // Latch for sensor or valve fault
END_VAR

METHOD Execute : BOOL
BEGIN
    // Initialize outputs if not already initialized
    IF NOT VALVE_OPEN_LATCH AND NOT FAULT_DETECTED THEN
        RELIEF_VALVE := FALSE; // Relief valve initially closed
        ALARM := FALSE;
    END_IF;

    // Check for sensor fault
    IF SensorFault THEN
        RELIEF_VALVE := TRUE; // Open relief valve on sensor fault
        ALARM := TRUE;
        FAULT_DETECTED := TRUE; // Latch the fault condition
    END_IF;

    // Check for valve malfunction
    IF ValveFault THEN
        RELIEF_VALVE := TRUE; // Open relief valve on valve malfunction
        ALARM := TRUE;
        FAULT_DETECTED := TRUE; // Latch the fault condition
    END_IF;

    // Check for overpressure condition
    IF PT_101 > HIGH_PRESSURE_LIMIT AND NOT VALVE_OPEN_LATCH AND NOT FAULT_DETECTED THEN
        RELIEF_VALVE := TRUE; // Open relief valve on overpressure
        ALARM := TRUE;
        VALVE_OPEN_LATCH := TRUE; // Latch the valve open condition
    END_IF;

    // Allow reset only if no faults and pressure drops below reset threshold
    IF PT_101 < RESET_THRESHOLD AND NOT SensorFault AND NOT ValveFault THEN
        VALVE_OPEN_LATCH := FALSE; // Clear the valve open latch
        FAULT_DETECTED := FALSE;   // Clear the fault latch
        RELIEF_VALVE := FALSE;     // Close relief valve
        ALARM := FALSE;
    END_IF;

    RETURN TRUE;
END_METHOD

END_PROGRAM
