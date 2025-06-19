FUNCTION_BLOCK FB_OverfillProtection
VAR_INPUT
    LevelSensor     : REAL;    // Measured level in vessel (e.g., %)
    SensorFault     : BOOL;    // TRUE if level sensor fails
    ValveFault      : BOOL;    // TRUE if inlet valve fails
    ManualReset     : BOOL;    // Optional manual reset button
END_VAR

VAR_OUTPUT
    INLET_VALVE     : BOOL;    // Valve control (TRUE = Open, FALSE = Closed)
    Alarm_Overfill  : BOOL;
    Alarm_Sensor    : BOOL;
    Alarm_Valve     : BOOL;
    InterlockActive : BOOL;
END_VAR

VAR
    HIGH_SETPOINT     : REAL := 90.0;    // Overfill threshold (%)
    RESET_THRESHOLD   : REAL := 80.0;    // Level to clear interlock (%)
    Latch_Overfill    : BOOL := FALSE;
END_VAR

// --- Sensor or Valve Fault Handling ---
IF SensorFault THEN
    Alarm_Sensor := TRUE;
    Latch_Overfill := TRUE;
END_IF;

IF ValveFault THEN
    Alarm_Valve := TRUE;
    Latch_Overfill := TRUE;
END_IF;

// --- Overfill Protection Logic ---
IF LevelSensor >= HIGH_SETPOINT AND NOT SensorFault THEN
    Latch_Overfill := TRUE;
    Alarm_Overfill := TRUE;
END_IF;

// --- Auto-Reset Logic ---
IF Latch_Overfill AND (LevelSensor <= RESET_THRESHOLD) AND NOT SensorFault AND NOT ValveFault THEN
    IF ManualReset THEN
        Latch_Overfill := FALSE;
        Alarm_Overfill := FALSE;
        Alarm_Sensor := FALSE;
        Alarm_Valve := FALSE;
    END_IF;
END_IF;

// --- Apply Interlock Action ---
InterlockActive := Latch_Overfill;

IF InterlockActive THEN
    INLET_VALVE := FALSE;    // Close valve
ELSE
    INLET_VALVE := TRUE;     // Open valve
END_IF;
