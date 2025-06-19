FUNCTION_BLOCK FB_FermentationTempControl
VAR_INPUT
    Enable         : BOOL;
    SetTemp        : REAL := 33.5;   // Target fermentation temp in °C
    MeasuredTemp   : REAL;           // From TIC-311
    DeltaT         : REAL := 1.0;    // Scan time in seconds
    HighLimit      : REAL := 36.0;   // Warning threshold
    TripLimit      : REAL := 37.0;   // Hard shutdown limit
END_VAR

VAR_OUTPUT
    CoolingValve   : REAL;           // 0.0 to 100.0 % control
    AlarmHighTemp  : BOOL;
    TripOverTemp   : BOOL;
END_VAR

VAR
    Error          : REAL;
    Integral       : REAL;
    Derivative     : REAL;
    PrevError      : REAL;
    Kp             : REAL := 5.0;    // Proportional gain
    Ki             : REAL := 0.5;    // Integral gain
    Kd             : REAL := 1.0;    // Derivative gain
    PID_Output     : REAL;
END_VAR

// PID logic only executes when control is enabled
IF Enable THEN
    Error := SetTemp - MeasuredTemp;
    Integral := Integral + Error * DeltaT;

    Derivative := (Error - PrevError) / DeltaT;
    PID_Output := Kp * Error + Ki * Integral + Kd * Derivative;

    // Clamp output between 0–100%
    IF PID_Output > 100.0 THEN
        PID_Output := 100.0;
    ELSIF PID_Output < 0.0 THEN
        PID_Output := 0.0;
    END_IF

    CoolingValve := PID_Output;

    // Over-temperature logic
    AlarmHighTemp := (MeasuredTemp >= HighLimit);
    TripOverTemp := (MeasuredTemp >= TripLimit);

    PrevError := Error;
ELSE
    CoolingValve := 0.0;
    AlarmHighTemp := FALSE;
    TripOverTemp := FALSE;
    Error := 0.0;
    Integral := 0.0;
    Derivative := 0.0;
END_IF
