FUNCTION_BLOCK FB_TurbineInterlocks
VAR_INPUT
    ExhaustTemp       : REAL; // °C
    TurbineSpeed      : REAL; // RPM
    CombustionPressure: REAL; // bar
    LubOilPressure    : REAL; // bar
    VibrationLevel    : REAL; // mm/s
    FlameDetected     : BOOL;
    FuelGasPressure   : REAL; // bar
    CoolingWaterFlow  : REAL; // m³/h
    SurgeDetected     : BOOL;
    EmergencyStop     : BOOL;
END_VAR

VAR_OUTPUT
    TripTurbine            : BOOL;
    CloseFuelValve         : BOOL;
    OpenPRV                : BOOL;
    StartBackupOilPump     : BOOL;
    Alarm_OverTemp         : BOOL;
    Alarm_Overspeed        : BOOL;
    Alarm_Overpressure     : BOOL;
    Alarm_LubeLow          : BOOL;
    Alarm_VibrationHigh    : BOOL;
    Alarm_FlameFailure     : BOOL;
    Alarm_FuelLow          : BOOL;
    Alarm_CoolingLow       : BOOL;
    Alarm_Surge            : BOOL;
    Alarm_EmergencyStop    : BOOL;
END_VAR

// Default values
TripTurbine := FALSE;
CloseFuelValve := FALSE;
OpenPRV := FALSE;
StartBackupOilPump := FALSE;

// Interlocks
IF ExhaustTemp > 650.0 THEN
    TripTurbine := TRUE;
    CloseFuelValve := TRUE;
    Alarm_OverTemp := TRUE;
END_IF;

IF TurbineSpeed > 110.0 THEN
    TripTurbine := TRUE;
    Alarm_Overspeed := TRUE;
END_IF;

IF CombustionPressure > 30.0 THEN
    OpenPRV := TRUE;
    Alarm_Overpressure := TRUE;
END_IF;

IF LubOilPressure < 1.5 THEN
    TripTurbine := TRUE;
    StartBackupOilPump := TRUE;
    Alarm_LubeLow := TRUE;
END_IF;

IF VibrationLevel > 12.0 THEN
    TripTurbine := TRUE;
    Alarm_VibrationHigh := TRUE;
END_IF;

IF NOT FlameDetected THEN
    CloseFuelValve := TRUE;
    Alarm_FlameFailure := TRUE;
END_IF;

IF FuelGasPressure < 5.0 THEN
    CloseFuelValve := TRUE;
    Alarm_FuelLow := TRUE;
END_IF;

IF CoolingWaterFlow < 30.0 THEN
    TripTurbine := TRUE;
    Alarm_CoolingLow := TRUE;
END_IF;

IF SurgeDetected THEN
    TripTurbine := TRUE;
    Alarm_Surge := TRUE;
END_IF;

IF EmergencyStop THEN
    TripTurbine := TRUE;
    CloseFuelValve := TRUE;
    Alarm_EmergencyStop := TRUE;
END_IF;
