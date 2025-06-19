VAR_INPUT
    StartBatch         : BOOL;
    CurrentTemp        : REAL;
    CurrentPressure    : REAL;
    CurrentVacuum      : REAL;
    BatchReset         : BOOL;
END_VAR

VAR CONSTANT
    Setpoint_Vacuum           : REAL := 200.0;    // mbar
    Setpoint_DeminWaterVol    : REAL := 300.0;    // L
    Setpoint_ReactionTempMin  : REAL := 55.0;     // °C
    Setpoint_ReactionTempMax  : REAL := 60.0;     // °C
    Setpoint_EndPressure      : REAL := 1.5;      // bar
    MaxReactionTime           : TIME := T#4h;
END_VAR


CASE PhaseStep OF

0: // IDLE
    IF StartBatch THEN
        StatusMessage := 'Batch Start: Begin Reactor Evacuation';
        PhaseStep := 1;
    END_IF

1: // Evacuate Reactor
    IF NOT EvacuationDone THEN
        EvacuateReactor();
        IF CurrentVacuum <= Setpoint_Vacuum THEN
            EvacuationDone := TRUE;
            PhaseStep := 2;
            StatusMessage := 'Evacuation Complete';
        END_IF
    END_IF

2: // Add Demineralized Water
    IF NOT WaterAdded THEN
        AddDemineralizedWater(Setpoint_DeminWaterVol);
        WaterAdded := TRUE;
        StatusMessage := 'Water Dosed';
        PhaseStep := 3;
    END_IF

3: // Polymerization Reaction Phase
    IF NOT ReactionStarted THEN
        StartPolymerization();
        ReactionTimer(IN := TRUE, PT := MaxReactionTime);
        ReactionStarted := TRUE;
        StatusMessage := 'Reaction Started';
    END_IF
    IF (CurrentPressure <= Setpoint_EndPressure) OR ReactionTimer.Q THEN
        ReactionTimer(IN := FALSE);
        PhaseStep := 4;
        StatusMessage := 'Reaction Complete – Proceeding to Decover';
    END_IF

4: // Decover Step
    IF NOT DecoverDone THEN
        ExecuteDecover();
        DecoverDone := TRUE;
        StatusMessage := 'Decover Completed';
        PhaseStep := 5;
    END_IF

5: // Drying Phase
    IF NOT DryingComplete THEN
        ExecuteDrying();
        DryingComplete := TRUE;
        StatusMessage := 'Drying Finished';
        PhaseStep := 6;
    END_IF

6: // Complete
    StatusMessage := 'Batch Completed Successfully';
    IF BatchReset THEN
        ResetAll();
        PhaseStep := 0;
    END_IF

END_CASE
