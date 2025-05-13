PROGRAM PLC_PRG
VAR
    // Constants
    HIGH_SETPOINT: REAL := 90.0;       // High-level threshold (%)
    RESET_THRESHOLD: REAL := 80.0;     // Reset level after shutdown (%)

    // Inputs
    LevelSensor: REAL := 0.0;          // Measured level from transmitter (%)
    SensorFault: BOOL := FALSE;        // TRUE if sensor signal is invalid or offline
    ValveFault: BOOL := FALSE;         // TRUE if valve feedback indicates failure

    // Outputs
    INLET_VALVE_OPEN: BOOL := TRUE;    // Inlet valve status (TRUE = Open, FALSE = Closed)
    HIGH_LEVEL_ALARM: BOOL := FALSE;   // Alarm to notify operator

    // Internal flags
    SHUTDOWN_LATCH: BOOL := FALSE;     // Latched state after high-level trip
END_VAR

// Evaluate alarm condition
HIGH_LEVEL_ALARM := LevelSensor > HIGH_SETPOINT;

// Main interlock logic
IF NOT SHUTDOWN_LATCH THEN
    IF LevelSensor > HIGH_SETPOINT OR SensorFault OR ValveFault THEN
        INLET_VALVE_OPEN := FALSE;
        SHUTDOWN_LATCH := TRUE;
    END_IF;
ELSE
    // Release latch only when level drops below reset threshold
    IF LevelSensor < RESET_THRESHOLD THEN
        SHUTDOWN_LATCH := FALSE;
    END_IF;
END_IF;

// Fail-safe override: if any fault persists, keep valve closed
IF SensorFault OR ValveFault THEN
    INLET_VALVE_OPEN := FALSE;
END_IF;
