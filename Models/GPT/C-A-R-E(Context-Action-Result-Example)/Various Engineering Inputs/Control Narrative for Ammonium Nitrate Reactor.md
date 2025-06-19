FUNCTION_BLOCK FB_AmmoniumNitrateReactorControl
VAR_INPUT
    Start         : BOOL;       // Start signal
    Stop          : BOOL;       // Stop signal
    Temp          : REAL;       // Reactor temperature (TIC-101)
    Pressure      : REAL;       // Reactor pressure (PIC-102)
    FlowAmmonia   : REAL;       // Ammonia flow (FT-201)
    FlowAcid      : REAL;       // Nitric acid flow (FT-202)
    PH_Value      : REAL;       // pH (AIC-104)
END_VAR

VAR_OUTPUT
    OpenESV_Amm   : BOOL;
    OpenESV_Acid  : BOOL;
    OpenSteam     : BOOL;
    OpenVent      : BOOL;
    AlarmTemp     : BOOL;
    AlarmPress    : BOOL;
    AlarmPH       : BOOL;
    AutoRunning   : BOOL;
END_VAR

VAR
    FlowRatio     : REAL;
    TempOK        : BOOL;
    PressOK       : BOOL;
    PH_OK         : BOOL;
END_VAR

// Calculate Flow Ratio (avoid division by zero)
IF FlowAcid > 0.01 THEN
    FlowRatio := FlowAmmonia / FlowAcid;
ELSE
    FlowRatio := 0.0;
END_IF;

// Check process conditions
TempOK  := (Temp >= 173.0) AND (Temp <= 177.0);        // ±2 °C tolerance
PressOK := (Pressure >= 4.6) AND (Pressure <= 5.0);     // ±0.2 bar tolerance
PH_OK   := (PH_Value >= 5.9) AND (PH_Value <= 6.5);     // pH range

// Alarm Conditions
AlarmTemp  := (Temp > 185.0);
AlarmPress := (Pressure > 5.2);
AlarmPH    := (PH_Value < 5.5) OR (PH_Value > 6.7);

// Emergency Shutdown
IF AlarmTemp OR AlarmPress THEN
    OpenESV_Amm := FALSE;
    OpenESV_Acid := FALSE;
    OpenSteam := FALSE;
    OpenVent := TRUE;
    AutoRunning := FALSE;
ELSIF Start AND NOT Stop THEN
    // Normal Operation
    IF TempOK AND PressOK AND PH_OK THEN
        OpenESV_Amm := TRUE;
        OpenESV_Acid := TRUE;
        OpenSteam := TRUE;
        OpenVent := FALSE;
        AutoRunning := TRUE;
    ELSE
        // Hold condition - sensors not within range
        OpenESV_Amm := FALSE;
        OpenESV_Acid := FALSE;
        OpenSteam := FALSE;
        OpenVent := FALSE;
        AutoRunning := FALSE;
    END_IF;
ELSE
    // Stop manually
    OpenESV_Amm := FALSE;
    OpenESV_Acid := FALSE;
    OpenSteam := FALSE;
    OpenVent := FALSE;
    AutoRunning := FALSE;
END_IF;
