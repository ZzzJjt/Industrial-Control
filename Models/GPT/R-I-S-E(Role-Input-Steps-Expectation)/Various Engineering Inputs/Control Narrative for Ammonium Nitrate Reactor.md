FUNCTION_BLOCK FB_AmmoniumNitrateReactorControl
VAR_INPUT
    Enable             : BOOL;
    MeasTemp           : REAL;     // °C
    MeasPressure       : REAL;     // bar
    MeasPH             : REAL;
    AmmoniaFlow        : REAL;     // kg/h
    AcidFlow           : REAL;     // kg/h
END_VAR

VAR_OUTPUT
    ValveNH3           : BOOL;
    ValveHNO3          : BOOL;
    HeaterOn           : BOOL;
    VentValve          : BOOL;
    Alarm              : BOOL;
    ESD                : BOOL;
END_VAR

VAR CONSTANT
    TempSetpoint       : REAL := 175.0;
    TempHiLimit        : REAL := 185.0;
    TempRangeMin       : REAL := 173.0;
    TempRangeMax       : REAL := 177.0;

    PressureSetpoint   : REAL := 4.8;
    PressureHiLimit    : REAL := 5.2;
    PressureRangeMin   : REAL := 4.6;
    PressureRangeMax   : REAL := 5.0;

    PHSetpoint         : REAL := 6.2;
    PHLoLimit          : REAL := 5.5;
    PHRangeMin         : REAL := 5.8;
    PHRangeMax         : REAL := 6.5;

    FlowRatioTarget    : REAL := 1.01;
    FlowRatioTolerance : REAL := 0.10; // ±10%
END_VAR

VAR
    FlowRatio          : REAL;
    TempOk             : BOOL;
    PressureOk         : BOOL;
    PHOk               : BOOL;
    RatioOk            : BOOL;
END_VAR

// Compute flow ratio
FlowRatio := AmmoniaFlow / AcidFlow;

// Validate conditions
TempOk     := (MeasTemp >= TempRangeMin) AND (MeasTemp <= TempRangeMax);
PressureOk := (MeasPressure >= PressureRangeMin) AND (MeasPressure <= PressureRangeMax);
PHOk       := (MeasPH >= PHRangeMin) AND (MeasPH <= PHRangeMax);
RatioOk    := ABS(FlowRatio - FlowRatioTarget) <= (FlowRatioTarget * FlowRatioTolerance);

// Alarm and ESD conditions
IF MeasTemp > TempHiLimit OR
   MeasPressure > PressureHiLimit OR
   MeasPH < PHLoLimit THEN
    ESD := TRUE;
    Alarm := TRUE;
ELSE
    ESD := FALSE;
    Alarm := NOT (TempOk AND PressureOk AND PHOk AND RatioOk);
END_IF;

// Reactor operation logic
IF Enable AND NOT ESD THEN
    HeaterOn := MeasTemp < TempSetpoint;
    ValveNH3 := RatioOk AND PHOk;
    ValveHNO3 := TRUE;
    VentValve := MeasPressure > PressureSetpoint;
ELSE
    HeaterOn := FALSE;
    ValveNH3 := FALSE;
    ValveHNO3 := FALSE;
    VentValve := TRUE; // vent on ESD
END_IF;
