// === Variable Declarations ===
VAR
    // Phase Control
    ReactionStep : INT := 0; // 0=Idle, 1=Heating, 2=Mixing, 3=Holding, 4=Complete

    // Parameters (input from recipe)
    Target_Temp : REAL := 90.0;         // °C
    Mixing_Speed : REAL := 120.0;       // RPM
    Hold_Time : TIME := T#15m;          // Reaction hold duration

    // Process I/O
    Current_Temp : REAL;                // From temperature sensor
    Current_Speed : REAL;               // From mixer
    Heater_On : BOOL := FALSE;
    Mixer_On : BOOL := FALSE;

    // Timers
    HeatTimer : TON;
    HoldTimer : TON;

    // State Flags
    Temp_Ready : BOOL := FALSE;
    Mixing_Started : BOOL := FALSE;
    Hold_Started : BOOL := FALSE;
    Reaction_Complete : BOOL := FALSE;
END_VAR

// === StartHeating Method ===
IF ReactionStep = 1 THEN
    Heater_On := TRUE;

    IF Current_Temp >= Target_Temp THEN
        Temp_Ready := TRUE;
        ReactionStep := 2;
    END_IF;
END_IF

// === StartMixing Method ===
IF ReactionStep = 2 THEN
    IF NOT Mixing_Started THEN
        Mixer_On := TRUE;
        Current_Speed := Mixing_Speed;
        Mixing_Started := TRUE;
    END_IF;

    // Assume mixing stabilization check (could add delay or feedback)
    ReactionStep := 3;
END_IF

// === HoldReaction Method ===
IF ReactionStep = 3 THEN
    IF NOT Hold_Started THEN
        HoldTimer(IN := TRUE, PT := Hold_Time);
        Hold_Started := TRUE;
    END_IF;

    IF HoldTimer.Q THEN
        Reaction_Complete := TRUE;
        Heater_On := FALSE;
        Mixer_On := FALSE;
        ReactionStep := 4;
    END_IF;
END_IF
