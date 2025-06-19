VAR
    // Batch step state machine
    ReactionStep : INT := 0;

    // Timers
    HeatTimer : TON;
    MixTimer : TON;
    HoldTimer : TON;

    // Timer control triggers
    HeatStart : BOOL := FALSE;
    MixStart : BOOL := FALSE;
    HoldStart : BOOL := FALSE;

    // Process parameters
    Target_Temp : REAL := 85.0;     // °C
    Temp_Tolerance : REAL := 2.0;   // ± °C
    Reaction_Time : TIME := T#10m;
    Heat_Time : TIME := T#5m;
    Mix_Time : TIME := T#3m;

    // Inputs
    Reactor_Temp : REAL;            // °C
    Mixing_Ready : BOOL;
    Heating_Ready : BOOL;

    // Outputs
    Heater_On : BOOL := FALSE;
    Mixer_On : BOOL := FALSE;
    Reaction_Complete : BOOL := FALSE;
END_VAR

// REACTION STEP SEQUENCE
CASE ReactionStep OF
    0: // Step B.2.1: Start Heating
        StartHeating();
        HeatStart := TRUE;
        HeatTimer(IN := HeatStart, PT := Heat_Time);

        IF HeatTimer.Q AND WithinTempTarget() THEN
            HeatStart := FALSE;
            HeatTimer(IN := FALSE);
            ReactionStep := 1;
        END_IF

    1: // Step B.2.2: Start Mixing
        StartMixing();
        MixStart := TRUE;
        MixTimer(IN := MixStart, PT := Mix_Time);

        IF MixTimer.Q THEN
            MixStart := FALSE;
            MixTimer(IN := FALSE);
            ReactionStep := 2;
        END_IF

    2: // Step B.2.3: Hold Reaction
        HoldReaction();
        HoldStart := TRUE;
        HoldTimer(IN := HoldStart, PT := Reaction_Time);

        IF HoldTimer.Q THEN
            HoldStart := FALSE;
            HoldTimer(IN := FALSE);
            StopAllOperations();
            Reaction_Complete := TRUE;
            ReactionStep := 3;
        END_IF

    3: // Complete
        ; // Do nothing. Await external step reset.
END_CASE

// Heating Phase Method
METHOD StartHeating
    Heater_On := TRUE;
END_METHOD

// Mixing Phase Method
METHOD StartMixing
    IF Heating_Ready THEN
        Mixer_On := TRUE;
    END_IF
END_METHOD

// Holding Phase Method
METHOD HoldReaction
    Heater_On := TRUE;
    Mixer_On := TRUE;
END_METHOD

// Shutdown Phase Method
METHOD StopAllOperations
    Heater_On := FALSE;
    Mixer_On := FALSE;
END_METHOD

// Temperature Check Utility
METHOD WithinTempTarget : BOOL
    WithinTempTarget := ABS(Reactor_Temp - Target_Temp) <= Temp_Tolerance;
END_METHOD
