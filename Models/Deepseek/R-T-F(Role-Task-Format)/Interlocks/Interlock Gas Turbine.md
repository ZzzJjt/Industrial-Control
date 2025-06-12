FUNCTION_BLOCK FB_OvertemperatureInterlock
VAR_INPUT
    Temperature : REAL; // Input temperature value
END_VAR

VAR_OUTPUT
    ShutdownTurbine : BOOL; // Output to shutdown turbine
    ActivateAlarm : BOOL; // Output to activate alarm
END_VAR

CONST
    MAX_TEMPERATURE : REAL := 700.0; // Maximum allowable temperature in °C
END_CONST

METHOD Execute : BOOL
BEGIN
    IF Temperature > MAX_TEMPERATURE THEN
        ShutdownTurbine := TRUE;
        ActivateAlarm := TRUE;
    ELSE
        ShutdownTurbine := FALSE;
        ActivateAlarm := FALSE;
    END_IF;

    RETURN TRUE;
END_METHOD

END_FUNCTION_BLOCK

FUNCTION_BLOCK FB_OverspeedInterlock
VAR_INPUT
    Speed : REAL; // Input speed value
END_VAR

VAR_OUTPUT
    EmergencyStop : BOOL; // Output to emergency stop
    EngageBrakes : BOOL; // Output to engage brakes
END_VAR

CONST
    MAX_SPEED : REAL := 3500.0; // Maximum allowable speed in RPM
END_CONST

METHOD Execute : BOOL
BEGIN
    IF Speed > MAX_SPEED THEN
        EmergencyStop := TRUE;
        EngageBrakes := TRUE;
    ELSE
        EmergencyStop := FALSE;
        EngageBrakes := FALSE;
    END_IF;

    RETURN TRUE;
END_METHOD

END_FUNCTION_BLOCK

FUNCTION_BLOCK FB_LowLubricationPressureInterlock
VAR_INPUT
    OilPressure : REAL; // Input oil pressure value
END_VAR

VAR_OUTPUT
    ShutdownTurbine : BOOL; // Output to shutdown turbine
    ActivateAlarm : BOOL; // Output to activate alarm
END_VAR

CONST
    MIN_OIL_PRESSURE : REAL := 0.1; // Minimum allowable oil pressure in MPa
END_CONST

METHOD Execute : BOOL
BEGIN
    IF OilPressure < MIN_OIL_PRESSURE THEN
        ShutdownTurbine := TRUE;
        ActivateAlarm := TRUE;
    ELSE
        ShutdownTurbine := FALSE;
        ActivateAlarm := FALSE;
    END_IF;

    RETURN TRUE;
END_METHOD

END_FUNCTION_BLOCK
