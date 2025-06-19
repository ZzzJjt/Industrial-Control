VAR
    Step : INT := 4; // Current phase of shutdown

    // Furnace
    CurrentTemp : REAL;
    TempTarget : REAL := 400.0;

    // Gas Flow
    GasFlowSetpoint : REAL;
    GasFlowCurrent : REAL;

    // Oxygen
    O2FlowSetpoint : REAL;
    O2FlowCurrent : REAL;

    // Timers
    ShutdownTimer : TON;
    GasRampTimer : TON;

    // Interlocks
    SafeTempReached : BOOL := FALSE;
    GasRampComplete : BOOL := FALSE;
    ShutdownComplete : BOOL := FALSE;
END_VAR


FUNCTION_BLOCK FB_GasRampDown
VAR_INPUT
    StartTime : TIME; // e.g., T#0s
    Duration : TIME := T#12h;
END_VAR
VAR_OUTPUT
    Setpoint : REAL;
END_VAR
VAR
    ElapsedTime : TIME;
    PercentRemaining : REAL;
END_VAR

ElapsedTime := TIME() - StartTime;
IF ElapsedTime < Duration THEN
    PercentRemaining := 1.0 - (REAL_TO_TIME(ElapsedTime) / REAL_TO_TIME(Duration));
    Setpoint := MAX(0.0, MIN(1.0, PercentRemaining)); // Clamp between 0 and 1
ELSE
    Setpoint := 0.0;
END_IF

FUNCTION_BLOCK FB_OxygenAdjust
VAR_INPUT
    FuelFlow : REAL;        // Actual gas flow in Nm³/h
    Temp : REAL;            // Reactor temp in °C
END_VAR
VAR_OUTPUT
    O2Setpoint : REAL;
    Alarm : BOOL;
END_VAR
VAR CONSTANT
    RATIO : REAL := 2.5;    // Target fuel-to-air ratio
    O2_MAX : REAL := 500.0; // Max oxygen flow Nm³/h
    O2_MIN : REAL := 20.0;
END_VAR

O2Setpoint := FuelFlow * RATIO;

// Safety limits
IF O2Setpoint > O2_MAX THEN
    O2Setpoint := O2_MAX;
    Alarm := TRUE;
ELSIF O2Setpoint < O2_MIN THEN
    O2Setpoint := O2_MIN;
    Alarm := TRUE;
ELSE
    Alarm := FALSE;
END_IF
