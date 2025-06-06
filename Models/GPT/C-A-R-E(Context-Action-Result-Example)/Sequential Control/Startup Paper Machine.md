FUNCTION_BLOCK FB_RampLinear
VAR_INPUT
    Start      : BOOL;
    StartValue : REAL;
    EndValue   : REAL;
    Duration   : TIME;
END_VAR
VAR_OUTPUT
    Output     : REAL;
    Done       : BOOL;
END_VAR
VAR
    Timer      : TON;
    Slope      : REAL;
END_VAR

Timer(IN := Start, PT := Duration);

IF Start AND NOT Done THEN
    Slope := (EndValue - StartValue) / TO_REAL(Duration / T#1s);
    Output := StartValue + Slope * TO_REAL(Timer.ET / T#1s);
    IF Timer.Q THEN
        Output := EndValue;
        Done := TRUE;
    END_IF
END_IF

VAR
    Step                  : INT := 0;
    StartCommand          : BOOL;
    AllDrivesReady        : BOOL;
    SafetyOK              : BOOL;
    WebTransportSynced    : BOOL;

    NipRamp               : FB_RampLinear;
    SpeedRamp             : FB_RampLinear;

    NipPressure           : REAL := 0.0;
    RollSpeed             : REAL := 0.0;
    PressRollTemperature  : REAL := 0.0;

    TargetPressure        : REAL := 250.0;       // kN/m
    TargetSpeed           : REAL := 500.0;       // m/min
    TargetTemp            : REAL := 85.0;        // °C
END_VAR

CASE Step OF

0: // Safety Pre-Check
    IF StartCommand AND SafetyOK AND AllDrivesReady THEN
        Step := 1;
    END_IF

1: // Start Web Transport System
    WebTransportSynced := TRUE; // Assume a feedback signal
    IF WebTransportSynced THEN
        Step := 2;
    END_IF

2: // Ramp-Up Roll Speed
    SpeedRamp(Start := TRUE, StartValue := 50.0, EndValue := TargetSpeed, Duration := T#5m);
    RollSpeed := SpeedRamp.Output;
    IF SpeedRamp.Done THEN
        Step := 3;
    END_IF

3: // Ramp-Up Nip Pressure
    NipRamp(Start := TRUE, StartValue := 0.0, EndValue := TargetPressure, Duration := T#10m);
    NipPressure := NipRamp.Output;
    IF NipRamp.Done THEN
        Step := 4;
    END_IF

4: // Enforce Roll Conditioning Temperature
    IF PressRollTemperature >= TargetTemp AND PressRollTemperature <= (TargetTemp + 5.0) THEN
        Step := 5;
    END_IF

5: // Press Section Ready
    // All systems ramped up and synchronized
    // Final status set or production flag raised here
END_CASE
