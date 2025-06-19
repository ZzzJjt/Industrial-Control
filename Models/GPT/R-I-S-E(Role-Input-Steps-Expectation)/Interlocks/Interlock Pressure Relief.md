FUNCTION_BLOCK FB_PressureReliefInterlock
VAR_INPUT
    PT_101         : REAL;     // Pressure reading (bar)
    SensorFault    : BOOL;     // TRUE if pressure transmitter has fault
    ValveFault     : BOOL;     // TRUE if relief valve has fault
    ManualReset    : BOOL;     // Manual reset input (e.g., button)
END_VAR

VAR_OUTPUT
    RELIEF_VALVE   : BOOL;     // TRUE = Valve open, FALSE = Valve closed
    Alarm_OverPressure : BOOL; // Indicates overpressure condition
    Alarm_Sensor   : BOOL;     // Indicates sensor fault
    Alarm_Valve    : BOOL;     // Indicates valve fault
    InterlockActive: BOOL;     // Latch flag for interlock activation
END_VAR

VAR CONSTANT
    HIGH_LIMIT     : REAL := 15.0;  // Overpressure limit in bar
    RESET_LIMIT    : REAL := 12.0;  // Reset threshold in bar
END_VAR

VAR
    SHUTDOWN_LATCH : BOOL := FALSE;
END_VAR

// --- Fault Detection ---
IF SensorFault THEN
    Alarm_Sensor := TRUE;
    SHUTDOWN_LATCH := TRUE;
END_IF;

IF ValveFault THEN
    Alarm_Valve := TRUE;
    SHUTDOWN_LATCH := TRUE;
END_IF;

// --- Overpressure Detection ---
IF PT_101 > HIGH_LIMIT AND NOT SensorFault THEN
    SHUTDOWN_LATCH := TRUE;
    Alarm_OverPressure := TRUE;
END_IF;

// --- Reset Logic ---
IF SHUTDOWN_LATCH AND PT_101 < RESET_LIMIT AND NOT SensorFault AND NOT ValveFault THEN
    IF ManualReset THEN
        SHUTDOWN_LATCH := FALSE;
        Alarm_OverPressure := FALSE;
        Alarm_Sensor := FALSE;
        Alarm_Valve := FALSE;
    END_IF;
END_IF;

// --- Interlock Output ---
InterlockActive := SHUTDOWN_LATCH;

IF InterlockActive THEN
    RELIEF_VALVE := TRUE;   // Open relief valve
ELSE
    RELIEF_VALVE := FALSE;  // Normal closed state
END_IF;
