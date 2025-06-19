(* === Pressure Relief Interlock System === *)

VAR_INPUT
    PT_VALUE        : REAL;   // Current pressure reading (bar)
    SENSOR_VALID    : BOOL;   // TRUE if the pressure sensor is working properly
    VALVE_FEEDBACK  : BOOL;   // TRUE if the valve is physically confirmed open
END_VAR

VAR_OUTPUT
    RELIEF_VALVE    : BOOL := FALSE;  // TRUE = Open, FALSE = Closed
    SHUTDOWN_LATCH  : BOOL := FALSE;  // Latches if overpressure or sensor fails
    SENSOR_FAULT    : BOOL := FALSE;  // TRUE if sensor is invalid
    VALVE_FAULT     : BOOL := FALSE;  // TRUE if valve did not respond as expected
    ALARM           : BOOL := FALSE;  // Operator alarm flag
END_VAR

VAR CONSTANT
    PRESSURE_HIGH_LIMIT  : REAL := 15.0;   // Pressure to open relief valve (bar)
    PRESSURE_RESET_LIMIT : REAL := 12.0;   // Pressure to close relief valve (bar)
END_VAR

// === Sensor Fault Detection ===
IF NOT SENSOR_VALID THEN
    SENSOR_FAULT := TRUE;
    SHUTDOWN_LATCH := TRUE;  // Trigger protection if sensor is faulty
ELSE
    SENSOR_FAULT := FALSE;
END_IF

// === Overpressure Protection Logic ===
IF SENSOR_VALID THEN
    IF PT_VALUE >= PRESSURE_HIGH_LIMIT THEN
        SHUTDOWN_LATCH := TRUE;
    ELSIF (PT_VALUE <= PRESSURE_RESET_LIMIT) AND (NOT SENSOR_FAULT) THEN
        SHUTDOWN_LATCH := FALSE;
    END_IF
END_IF

// === Relief Valve Control ===
IF SHUTDOWN_LATCH THEN
    RELIEF_VALVE := TRUE;   // Open the valve
ELSE
    RELIEF_VALVE := FALSE;  // Close the valve
END_IF

// === Valve Fault Detection ===
IF RELIEF_VALVE AND NOT VALVE_FEEDBACK THEN
    VALVE_FAULT := TRUE;
    ALARM := TRUE;
ELSIF NOT RELIEF_VALVE AND VALVE_FEEDBACK THEN
    VALVE_FAULT := TRUE;
    ALARM := TRUE;
ELSE
    VALVE_FAULT := FALSE;
    ALARM := FALSE;
END_IF
