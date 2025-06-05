PROGRAM UreaReactionControl
VAR
    // --- Inputs ---
    stAmmoniaValve       : BOOL;     // Status of ammonia valve (sensor)
    stCO2Valve           : BOOL;     // Status of CO₂ valve (sensor)
    rCurrentPressure     : REAL;     // Reactor pressure in bar
    rCurrentTemperature  : REAL;     // Reactor temperature in °C
    CURRENT_TIME         : TIME;     // System time (external RTC)

    // --- Outputs ---
    stAmmoniaValveControl : BOOL;    // Command to open/close ammonia valve
    stCO2ValveControl     : BOOL;    // Command to open/close CO₂ valve
    stReactionFinished    : BOOL;    // Process completion flag

    // --- Parameters ---
    rTargetPressure       : REAL := 175.0;
    rPressureTolerance    : REAL := 5.0;
    rTargetTemperature    : REAL := 185.0;
    rTemperatureTolerance : REAL := 2.0;
    tReactionDuration     : TIME := T#30m;

    // --- Internals ---
    tReactionStart        : TIME;
    bTimerStarted         : BOOL := FALSE;
    bConditionsOK         : BOOL := FALSE;
    tElapsed              : TIME;
END_VAR

// Step 1: Material Intake
IF NOT bTimerStarted THEN
    stAmmoniaValveControl := TRUE;
    stCO2ValveControl := TRUE;

    IF stAmmoniaValve AND stCO2Valve THEN
        tReactionStart := CURRENT_TIME;
        bTimerStarted := TRUE;
    END_IF
END_IF

// Step 2: Monitor Conditions
IF bTimerStarted AND NOT stReactionFinished THEN
    bConditionsOK := 
        ABS(rCurrentPressure - rTargetPressure) <= rPressureTolerance AND
        ABS(rCurrentTemperature - rTargetTemperature) <= rTemperatureTolerance;

    tElapsed := CURRENT_TIME - tReactionStart;

    IF bConditionsOK AND tElapsed >= tReactionDuration THEN
        stAmmoniaValveControl := FALSE;
        stCO2ValveControl := FALSE;
        stReactionFinished := TRUE;
    END_IF
END_IF
