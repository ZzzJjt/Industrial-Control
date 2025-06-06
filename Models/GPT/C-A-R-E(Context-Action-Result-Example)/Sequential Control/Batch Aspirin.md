VAR_INPUT
    StartBatch        : BOOL;
    ReactionTemp      : REAL := 85.0;
    ReactionPressure  : REAL := 1.5;
    ReactionTime      : TIME := T#25m;
    MixSpeed          : INT := 300;
    DryingTemp        : REAL := 90.0;
    DryingTime        : TIME := T#40m;
    CurrentTemp       : REAL;
    CurrentPressure   : REAL;
END_VAR

VAR_OUTPUT
    BatchComplete     : BOOL;
    StatusMessage     : STRING[100];
END_VAR

VAR
    PhaseStep         : INT := 0;
    ReactionTimer     : TON;
    DryingTimer       : TON;
END_VAR

CASE PhaseStep OF

0: // Wait for Start
    IF StartBatch THEN
        StatusMessage := 'Initializing Reaction Phase';
        PhaseStep := 1;
    END_IF

1: // Heating for Reaction
    StatusMessage := 'Heating to Reaction Temperature';
    IF CurrentTemp >= ReactionTemp THEN
        PhaseStep := 2;
    END_IF

2: // Mixing Start
    StatusMessage := 'Starting Mixing at Speed';
    (* Hardware call: SetMixerSpeed(MixSpeed); *)
    PhaseStep := 3;

3: // Hold Reaction
    StatusMessage := 'Holding Reaction Conditions';
    IF CurrentTemp >= ReactionTemp AND CurrentPressure >= ReactionPressure THEN
        ReactionTimer(IN := TRUE, PT := ReactionTime);
        IF ReactionTimer.Q THEN
            ReactionTimer(IN := FALSE);
            PhaseStep := 4;
        END_IF
    END_IF

4: // Crystallization (placeholder)
    StatusMessage := 'Initiating Crystallization';
    // Actual logic may include cooling, seeding, etc.
    PhaseStep := 5;

5: // Heating for Drying
    StatusMessage := 'Heating to Drying Temperature';
    IF CurrentTemp >= DryingTemp THEN
        PhaseStep := 6;
    END_IF

6: // Drying Process
    StatusMessage := 'Drying in Progress';
    DryingTimer(IN := TRUE, PT := DryingTime);
    IF DryingTimer.Q THEN
        DryingTimer(IN := FALSE);
        PhaseStep := 7;
    END_IF

7: // Batch Complete
    StatusMessage := 'Batch Complete';
    BatchComplete := TRUE;

END_CASE
