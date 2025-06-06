VAR
    Phase            : STRING := 'Idle';
    Timer            : TON;
    
    ReactorTemp      : REAL := 25.0;
    ReactorPressure  : REAL := 1.0;

    SetTemp          : REAL := 180.0; // °C
    SetPressure      : REAL := 140.0; // bar
    ReactionTime     : TIME := T#30m;

    Heater           : BOOL := FALSE;
    PressureValve    : BOOL := FALSE;
    CoolingValve     : BOOL := FALSE;

    StartReaction    : BOOL := FALSE;
    ProcessComplete  : BOOL := FALSE;
END_VAR

CASE Phase OF

'Idle':
    IF StartReaction THEN
        Phase := 'HeatUp';
    END_IF

'HeatUp':
    Heater := TRUE;
    IF ReactorTemp >= SetTemp THEN
        Heater := FALSE;
        Timer(IN := TRUE, PT := T#2m); // Stabilization delay
        Phase := 'TempHoldDelay';
    END_IF

'TempHoldDelay':
    IF Timer.Q THEN
        Timer(IN := FALSE);
        Phase := 'PressureHold';
    END_IF

'PressureHold':
    PressureValve := TRUE;
    IF ReactorPressure >= SetPressure THEN
        Timer(IN := TRUE, PT := ReactionTime);
        Phase := 'ReactionRunning';
    END_IF

'ReactionRunning':
    IF Timer.Q THEN
        Timer(IN := FALSE);
        PressureValve := FALSE;
        Phase := 'CoolDown';
    END_IF

'CoolDown':
    CoolingValve := TRUE;
    IF ReactorTemp <= 80.0 THEN // Cool to 80°C threshold
        CoolingValve := FALSE;
        Phase := 'Complete';
    END_IF

'Complete':
    ProcessComplete := TRUE;
    IF NOT StartReaction THEN // Wait for operator to reset start
        Phase := 'Idle';
        ProcessComplete := FALSE;
    END_IF

END_CASE
