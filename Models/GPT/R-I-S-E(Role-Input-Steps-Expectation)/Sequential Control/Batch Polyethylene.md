VAR
    Step : INT := 0;           // Current stage (0–7)
    Phase : INT := 0;          // Phase within current stage
    Done : BOOL := FALSE;      // End of batch

    // Control Outputs
    Valve_Feedstock : BOOL;
    Mixer_Catalyst : BOOL;
    Heater_Reactor : BOOL;
    Quench_Injector : BOOL;
    Dryer_Unit : BOOL;
    Extruder : BOOL;
    PelletCutter : BOOL;
    Sampler : BOOL;
    Packer : BOOL;
    StorageConveyor : BOOL;

    // Timers
    T_Phase : TON;
    PhaseTimerActive : BOOL := FALSE;

    // Process Conditions
    Temp : REAL;
    Temp_SP : REAL := 180.0;       // Polymerization temp
    Cool_SP : REAL := 80.0;        // Quenching temp
    Dry_SP : REAL := 100.0;
    ReactionTime : TIME := T#30m;
    DryTime : TIME := T#20m;
    QuenchTime : TIME := T#5m;
    PelletTime : TIME := T#3m;

    // Flags
    PhaseDone : BOOL;
END_VAR


IF NOT T_Phase.IN AND PhaseTimerActive THEN
    T_Phase(IN := TRUE, PT := ReactionTime);
END_IF

IF T_Phase.Q THEN
    PhaseDone := TRUE;
    T_Phase(IN := FALSE);      // Reset
    PhaseTimerActive := FALSE;
END_IF

IF Step = 1 THEN
    CASE Phase OF
        0: // Feedstock dosing
            Valve_Feedstock := TRUE;
            PhaseTimerActive := TRUE;
            T_Phase.PT := T#2m;
            Phase := 1;

        1:
            IF PhaseDone THEN
                Valve_Feedstock := FALSE;
                PhaseDone := FALSE;
                Phase := 2;
            END_IF

        2: // Catalyst Mixing
            Mixer_Catalyst := TRUE;
            PhaseTimerActive := TRUE;
            T_Phase.PT := T#1m;
            Phase := 3;

        3:
            IF PhaseDone THEN
                Mixer_Catalyst := FALSE;
                PhaseDone := FALSE;
                Phase := 0;
                Step := 2;
            END_IF
    END_CASE
END_IF
