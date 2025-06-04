(* === Overfill Protection Interlock === *)

VAR_INPUT
    LEVEL_SENSOR   : REAL;   // Tank level (%) from 0.0 to 100.0
    SENSOR_VALID   : BOOL;   // TRUE if sensor data is valid
    VALVE_FEEDBACK : BOOL;   // TRUE if valve is physically open
END_VAR

VAR_OUTPUT
    INLET_VALVE       : BOOL := TRUE;   // TRUE = Open, FALSE = Closed
    SHUTDOWN_LATCH    : BOOL := FALSE;  // Latched shutdown state
    SENSOR_FAULT      : BOOL := FALSE;  // Sensor failure detected
    VALVE_FAULT       : BOOL := FALSE;  // Valve failed to respond
END_VAR

VAR CONSTANT
    LEVEL_HIGH_LIMIT  : REAL := 95.0;   // High-level shutdown threshold (%)
    LEVEL_RESET_LIMIT : REAL := 85.0;   // Reset threshold to reopen valve (%)
END_VAR

VAR
    ValveCmdIssued : BOOL := FALSE;     // Internal flag for valve action tracking
END_VAR

// === Sensor Fault Check ===
IF NOT SENSOR_VALID THEN
    SENSOR_FAULT := TRUE;
    SHUTDOWN_LATCH := TRUE;
ELSE
    SENSOR_FAULT := FALSE;
END_IF

// === Overfill Protection Logic ===
IF SENSOR_VALID THEN
    IF LEVEL_SENSOR >= LEVEL_HIGH_LIMIT THEN
        SHUTDOWN_LATCH := TRUE;
    ELSIF (LEVEL_SENSOR <= LEVEL_RESET_LIMIT) AND (NOT SENSOR_FAULT) THEN
        SHUTDOWN_LATCH := FALSE;
    END_IF
END_IF

// === Valve Control ===
IF SHUTDOWN_LATCH THEN
    INLET_VALVE := FALSE;
ELSE
    INLET_VALVE := TRUE;
END_IF

// === Valve Fault Detection ===
IF INLET_VALVE AND NOT VALVE_FEEDBACK THEN
    VALVE_FAULT := TRUE;  // Commanded open, but valve still closed
ELSIF NOT INLET_VALVE AND VALVE_FEEDBACK THEN
    VALVE_FAULT := TRUE;  // Commanded closed, but valve still open
ELSE
    VALVE_FAULT := FALSE;
END_IF
