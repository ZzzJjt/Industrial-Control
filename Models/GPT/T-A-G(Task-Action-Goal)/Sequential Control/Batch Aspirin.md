VAR
    // Process step control
    ReactionStep : INT := 0;

    // Timers
    HeatTimer : TON;
    MixTimer : TON;
    ReactHoldTimer : TON;

    // Timer triggers
    HeatTrig : BOOL := FALSE;
    MixTrig : BOOL := FALSE;
    HoldTrig : BOOL := FALSE;

    // Process Parameters
    Temp_Target : REAL := 70.0;          // °C
    Temp_Tolerance : REAL := 2.0;
    Pressure_Max : REAL := 1.5;          // bar
    Reaction_Duration : TIME := T#20m;
    Heat_Time : TIME := T#5m;
    Mix_Time : TIME := T#3m;

    // Process Inputs
    Reactor_Temp : REAL;
    Reactor_Pressure : REAL;

    // Outputs
    Heater : BOOL := FALSE;
    Mixer : BOOL := FALSE;
    Reaction_Complete : BOOL := FALSE;
END_VAR

CASE ReactionStep OF
    0: // Start Heating
        StartHeating();
        HeatTrig := TRUE;
        HeatTimer(IN := HeatTrig, PT := Heat_Time);
        
        IF HeatTimer.Q AND WithinTempTarget() THEN
            HeatTrig := FALSE;
            HeatTimer(IN := FALSE);
            ReactionStep := 1;
        END_IF

    1: // Start Mixing
        StartMixing();
        MixTrig := TRUE;
        MixTimer(IN := MixTrig, PT := Mix_Time);
        
        IF MixTimer.Q THEN
            MixTrig := FALSE;
            MixTimer(IN := FALSE);
            ReactionStep := 2;
        END_IF

    2: // Hold Reaction
        HoldReaction();
        HoldTrig := TRUE;
        ReactHoldTimer(IN := HoldTrig, PT := Reaction_Duration);
        
        IF ReactHoldTimer.Q OR Overpressure() THEN
            HoldTrig := FALSE;
            ReactHoldTimer(IN := FALSE);
            StopAll();
            Reaction_Complete := TRUE;
            ReactionStep := 3;
        END_IF

    3:
        ; // Idle until next command
END_CASE

METHOD StartHeating
    Heater := TRUE;
END_METHOD

METHOD StartMixing
    Mixer := TRUE;
END_METHOD

METHOD HoldReaction
    Heater := TRUE;
    Mixer := TRUE;
END_METHOD

METHOD StopAll
    Heater := FALSE;
    Mixer := FALSE;
END_METHOD

METHOD WithinTempTarget : BOOL
    WithinTempTarget := ABS(Reactor_Temp - Temp_Target) <= Temp_Tolerance;
END_METHOD

METHOD Overpressure : BOOL
    Overpressure := Reactor_Pressure > Pressure_Max;
END_METHOD

VAR
    Dryer_Temp : REAL;
    Dryer_Timer : TON;
    DryingTrig : BOOL := FALSE;
    Drying_Duration : TIME := T#30m;
    DryingComplete : BOOL := FALSE;
END_VAR

METHOD RunDrying
    IF ABS(Dryer_Temp - 90.0) <= 1.0 THEN
        DryingTrig := TRUE;
        Dryer_Timer(IN := DryingTrig, PT := Drying_Duration);
        IF Dryer_Timer.Q THEN
            DryingTrig := FALSE;
            Dryer_Timer(IN := FALSE);
            DryingComplete := TRUE;
        END_IF
    END_IF
END_METHOD
