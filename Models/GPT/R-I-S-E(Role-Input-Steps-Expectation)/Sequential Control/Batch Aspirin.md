VAR
    // Global Recipe Parameters
    Temp_Setpoint : REAL := 80.0;          // Reaction temp °C
    Drying_Temp   : REAL := 90.0;          // °C
    Reaction_Hold : TIME := T#30m;
    Drying_Time   : TIME := T#45m;

    // Sensors and Actuators
    Temp_Reactor : REAL;
    Temp_Dryer : REAL;
    Heater_Reactor : BOOL := FALSE;
    Heater_Dryer : BOOL := FALSE;
    Mixer_Reactor : BOOL := FALSE;
    Valve_CentrifugeOut : BOOL := FALSE;

    // Phase Control
    BatchStep : INT := 0; // 0=Idle, 1=Reaction, 2=Crystallization, etc.
    ReactionTimer : TON;
    DryingTimer : TON;

    // Flags
    Reaction_Started : BOOL := FALSE;
    Reaction_Done : BOOL := FALSE;
    Drying_Started : BOOL := FALSE;
    Batch_Complete : BOOL := FALSE;
END_VAR

// === REACTION PHASE ===
IF BatchStep = 1 THEN
    // Start heating reactor
    Heater_Reactor := TRUE;
    IF Temp_Reactor >= Temp_Setpoint THEN
        // Start mixing & hold reaction
        Mixer_Reactor := TRUE;

        IF NOT Reaction_Started THEN
            ReactionTimer(IN := TRUE, PT := Reaction_Hold);
            Reaction_Started := TRUE;
        END_IF;

        IF ReactionTimer.Q THEN
            Heater_Reactor := FALSE;
            Mixer_Reactor := FALSE;
            Reaction_Done := TRUE;
            BatchStep := 2; // Proceed to crystallization
        END_IF;
    END_IF;
END_IF

// === CRYSTALLIZATION PHASE ===
IF BatchStep = 2 THEN
    // Assume passive cooling and monitoring; logic can be extended
    IF Temp_Reactor < 30.0 THEN // Wait until cool enough to crystallize
        BatchStep := 3;
    END_IF;
END_IF
