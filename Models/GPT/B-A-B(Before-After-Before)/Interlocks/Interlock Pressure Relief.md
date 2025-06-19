FUNCTION_BLOCK FB_PressureReliefProtection
VAR_INPUT
    PT_101        : REAL;     // Pressure transmitter reading (bar)
    HIGH_LIMIT    : REAL;     // High pressure threshold (e.g., 15 bar)
    RESET_LIMIT   : REAL;     // Reset threshold (e.g., 12 bar)
    SensorFault   : BOOL;     // TRUE if pressure sensor is faulty
    ValveFault    : BOOL;     // TRUE if relief valve is faulty
END_VAR

VAR_OUTPUT
    RELIEF_VALVE  : BOOL;     // TRUE = valve open
    RELIEF_LATCH  : BOOL;     // Indicates latch condition (overpressure)
    AlarmHigh     : BOOL;     // Alarm for high pressure
    AlarmSensor   : BOOL;     // Alarm for sensor fault
    AlarmValve    : BOOL;     // Alarm for valve fault
END_VAR

VAR
    FailSafe      : BOOL := FALSE; // Internal fail-safe trigger
END_VAR

// --- Set alarms ---
AlarmHigh   := PT_101 >= HIGH_LIMIT;
AlarmSensor := SensorFault;
AlarmValve  := ValveFault;

// --- Overpressure logic with latching behavior ---
IF PT_101 >= HIGH_LIMIT THEN
    RELIEF_LATCH := TRUE;
END_IF;

IF RELIEF_LATCH AND (PT_101 < RESET_LIMIT) THEN
    RELIEF_LATCH := FALSE;
END_IF;

// --- Fail-safe trigger if sensor or valve is faulty ---
FailSafe := SensorFault OR ValveFault;

// --- Final control decision ---
IF RELIEF_LATCH OR FailSafe THEN
    RELIEF_VALVE := TRUE;  // Open valve to relieve pressure
ELSE
    RELIEF_VALVE := FALSE; // Normal operation
END_IF;
