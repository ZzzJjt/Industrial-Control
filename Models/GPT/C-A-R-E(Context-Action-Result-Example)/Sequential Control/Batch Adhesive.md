FUNCTION_BLOCK FB_ReactionStep
VAR_INPUT
    Start           : BOOL;
    TargetTemp      : REAL := 85.0;
    MixSpeed        : INT := 600;
    HeatHoldTime    : TIME := T#10m;
    MixTime         : TIME := T#15m;
    HoldTime        : TIME := T#20m;
    CurrentTemp     : REAL;
END_VAR

VAR_OUTPUT
    Done            : BOOL;
    StepStatus      : STRING[50];
END_VAR

VAR
    Phase           : INT := 0;
    HeatingTimer    : TON;
    MixingTimer     : TON;
    HoldingTimer    : TON;
END_VAR


CASE Phase OF
0: // Wait for start trigger
    IF Start THEN
        StepStatus := 'Initializing Reaction Phase';
        Phase := 1;
    END_IF

1: // Start Heating
    StepStatus := 'Heating to Target Temperature';
    HeatingTimer(IN := CurrentTemp >= TargetTemp, PT := HeatHoldTime);
    IF HeatingTimer.Q THEN
        HeatingTimer(IN := FALSE);
        Phase := 2;
    END_IF

2: // Start Mixing
    StepStatus := 'Mixing at Constant Speed';
    (* Assume SetMixerSpeed(MixSpeed) is called in hardware layer *)
    MixingTimer(IN := TRUE, PT := MixTime);
    IF MixingTimer.Q THEN
        MixingTimer(IN := FALSE);
        Phase := 3;
    END_IF

3: // Hold Reaction
    StepStatus := 'Holding Reaction Conditions';
    HoldingTimer(IN := TRUE, PT := HoldTime);
    IF HoldingTimer.Q THEN
        HoldingTimer(IN := FALSE);
        Phase := 4;
    END_IF

4: // Complete
    StepStatus := 'Reaction Step Complete';
    Done := TRUE;
END_CASE
