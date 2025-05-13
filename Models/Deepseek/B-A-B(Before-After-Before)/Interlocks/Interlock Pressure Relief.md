PROGRAM PLC_PRG
VAR
    // Constants
    HIGH_LIMIT: REAL := 15.0;          // High-pressure threshold (bar)
    RESET_LIMIT: REAL := 12.0;         // Reset pressure after relief (bar)

    // Inputs
    PT_101: REAL := 0.0;               // Measured pressure from transmitter (bar)
    SensorFault: BOOL := FALSE;        // TRUE if sensor signal is invalid or offline
    ValveFault: BOOL := FALSE;         // TRUE if valve feedback indicates failure

    // Outputs
    RELIEF_VALVE_OPEN: BOOL := FALSE;  // Relief valve status (TRUE = Open, FALSE = Closed)
    OVERPRESSURE_ALARM: BOOL := FALSE; // Alarm to notify operator

    // Internal flags
    RELIEF_LATCH: BOOL := FALSE;       // Latched state after high-pressure trip
END_VAR

// Evaluate alarm condition
OVERPRESSURE_ALARM := PT_101 > HIGH_LIMIT;

// Main interlock logic
IF NOT RELIEF_LATCH THEN
    IF PT_101 > HIGH_LIMIT OR SensorFault THEN
        RELIEF_VALVE_OPEN := TRUE;
        RELIEF_LATCH := TRUE;
    END_IF;
ELSE
    // Release latch only when pressure drops below reset limit
    IF PT_101 < RESET_LIMIT THEN
        RELIEF_LATCH := FALSE;
        RELIEF_VALVE_OPEN := FALSE;
    END_IF;
END_IF;

// Fail-safe override: if any fault persists, keep valve open
IF SensorFault OR ValveFault THEN
    RELIEF_VALVE_OPEN := TRUE;
END_IF;
