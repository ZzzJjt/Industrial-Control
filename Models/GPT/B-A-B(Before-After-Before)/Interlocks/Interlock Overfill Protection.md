FUNCTION_BLOCK FB_OverfillProtection
VAR_INPUT
    LevelSensor     : REAL;     // Level sensor reading (e.g., % or cm)
    HIGH_SETPOINT   : REAL;     // High-level trip threshold
    RESET_THRESHOLD : REAL;     // Reset threshold (to prevent chattering)
    SensorFault     : BOOL;     // Level sensor fault flag
    ValveFault      : BOOL;     // Inlet valve fault flag
END_VAR

VAR_OUTPUT
    INLET_VALVE     : BOOL;     // Inlet valve control (TRUE = open, FALSE = closed)
    SHUTDOWN_LATCH  : BOOL;     // Indicates whether overfill shutdown is latched
    AlarmHighLevel  : BOOL;     // Alarm for high-level condition
    AlarmSensorFail : BOOL;     // Alarm for sensor failure
    AlarmValveFail  : BOOL;     // Alarm for valve fault
END_VAR

VAR
    ValveClosedDueToFault : BOOL := FALSE;
END_VAR

// --- Alarm conditions ---
AlarmHighLevel  := LevelSensor >= HIGH_SETPOINT;
AlarmSensorFail := SensorFault;
AlarmValveFail  := ValveFault;

// --- Latching logic for overfill shutdown ---
IF LevelSensor >= HIGH_SETPOINT THEN
    SHUTDOWN_LATCH := TRUE;
END_IF;

IF SHUTDOWN_LATCH AND (LevelSensor < RESET_THRESHOLD) THEN
    SHUTDOWN_LATCH := FALSE;
END_IF;

// --- Fail-safe logic in case of sensor or valve fault ---
ValveClosedDueToFault := SensorFault OR ValveFault;

// --- Final control decision for inlet valve ---
IF SHUTDOWN_LATCH OR ValveClosedDueToFault THEN
    INLET_VALVE := FALSE; // Close inlet valve
ELSE
    INLET_VALVE := TRUE;  // Allow normal operation
END_IF;
