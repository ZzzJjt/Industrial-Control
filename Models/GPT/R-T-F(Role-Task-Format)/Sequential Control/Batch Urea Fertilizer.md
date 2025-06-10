VAR
    // === Batch Phase States ===
    Phase_Main         : INT := 0;     // 0=Idle, 1=RawMatPrep ... 7=Packaging
    SubStep            : INT := 0;     // Step tracker within each phase

    // === Process Setpoints ===
    Temp_Reactor       : REAL := 130.0;
    Pressure_Reactor   : REAL := 10.5;
    Temp_Dryer         : REAL := 90.0;
    PelletCountTarget  : INT := 100000;

    // === Sensors and Status Flags ===
    Temp_Current       : REAL;
    Pressure_Current   : REAL;
    HopperLoaded       : BOOL;
    ReactantsCharged   : BOOL;
    ReactionComplete   : BOOL;
    QuenchComplete     : BOOL;
    DryingComplete     : BOOL;
    PelletizingDone    : BOOL;
    QC_Passed          : BOOL;
    PackagingReady     : BOOL;

    // === Output Controls ===
    HeaterOn           : BOOL;
    DryerOn            : BOOL;
    FeederMotor        : BOOL;
    QuenchValve        : BOOL;
    PelletizerMotor    : BOOL;
    ConveyorBelt       : BOOL;
    PackagingUnit      : BOOL;

    // === Timers ===
    PhaseTimer         : TON;
    TimerStart         : BOOL := FALSE;

    // === Counters & Internal Variables ===
    PelletCount        : INT := 0;
END_VAR

// === MAIN PHASE CONTROLLER ===
CASE Phase_Main OF

    // ----------------------------
    1: // RAW MATERIAL PREPARATION
    CASE SubStep OF
        0:
            LoadHopper(); // External method
            IF HopperLoaded THEN
                SubStep := 1;
            END_IF;
        1:
            ChargeReactants(); // External method
            IF ReactantsCharged THEN
                SubStep := 0;
                Phase_Main := 2; // Next: Polymerization
            END_IF;
    END_CASE;

    // ----------------------------
    2: // POLYMERIZATION
    CASE SubStep OF
        0:
            StartHeating(TargetTemp := Temp_Reactor);
            StartPressurizing(TargetPressure := Pressure_Reactor);
            IF Temp_Current >= Temp_Reactor AND Pressure_Current >= Pressure_Reactor THEN
                TimerStart := TRUE;
                PhaseTimer(IN := TimerStart, PT := T#30m); // Hold 30 min
                SubStep := 1;
            END_IF;
        1:
            IF PhaseTimer.Q THEN
                TimerStart := FALSE;
                HeaterOn := FALSE;
                ReactionComplete := TRUE;
                SubStep := 0;
                Phase_Main := 3; // Next: Quenching
            END_IF;
    END_CASE;

    // ----------------------------
    3: // QUENCHING
    CASE SubStep OF
        0:
            QuenchValve := TRUE;
            TimerStart := TRUE;
            PhaseTimer(IN := TimerStart, PT := T#5m);
            SubStep := 1;
        1:
            IF PhaseTimer.Q THEN
                QuenchValve := FALSE;
                TimerStart := FALSE;
                QuenchComplete := TRUE;
                SubStep := 0;
                Phase_Main := 4; // Next: Drying
            END_IF;
    END_CASE;

    // ----------------------------
    4: // DRYING
    CASE SubStep OF
        0:
            DryerOn := TRUE;
            TimerStart := TRUE;
            PhaseTimer(IN := TimerStart, PT := T#20m);
            SubStep := 1;
        1:
            IF PhaseTimer.Q THEN
                DryerOn := FALSE;
                TimerStart := FALSE;
                DryingComplete := TRUE;
                SubStep := 0;
                Phase_Main := 5; // Next: Pelletizing
            END_IF;
    END_CASE;

    // ----------------------------
    5: // PELLETIZING
    CASE SubStep OF
        0:
            BeginPelletizing(); // Start motor, sync with cutter
            PelletizerMotor := TRUE;
            IF PelletCount >= PelletCountTarget THEN
                PelletizerMotor := FALSE;
                PelletizingDone := TRUE;
                SubStep := 0;
                Phase_Main := 6; // Next: Quality Control
            END_IF;
    END_CASE;

    // ----------------------------
    6: // QUALITY CONTROL
    CASE SubStep OF
        0:
            StartQCAnalysis(); // e.g., weight, density, purity
            IF QC_Passed THEN
                SubStep := 0;
                Phase_Main := 7; // Next: Packaging
            END_IF;
    END_CASE;

    // ----------------------------
    7: // PACKAGING AND STORAGE
    CASE SubStep OF
        0:
            ConveyorBelt := TRUE;
            PackagingUnit := TRUE;
            IF PackagingReady THEN
                ConveyorBelt := FALSE;
                PackagingUnit := FALSE;
                SubStep := 0;
                Phase_Main := 0; // End of Batch
            END_IF;
    END_CASE;

END_CASE;

METHOD LoadHopper
// Logic to open valves or run feeders
BEGIN
    FeederMotor := TRUE;
    IF LevelSensorOK THEN
        FeederMotor := FALSE;
        HopperLoaded := TRUE;
    END_IF;
END_METHOD

METHOD ChargeReactants
// Sequence-controlled dosing of ethylene and catalysts
BEGIN
    OpenValve("Ethylene");
    OpenValve("Catalyst");
    IF ReactantSensorsOK THEN
        ReactantsCharged := TRUE;
    END_IF;
END_METHOD

METHOD StartHeating
VAR_INPUT TargetTemp : REAL;
END_VAR
BEGIN
    IF Temp_Current < TargetTemp THEN
        HeaterOn := TRUE;
    ELSE
        HeaterOn := FALSE;
    END_IF;
END_METHOD

METHOD BeginPelletizing
BEGIN
    PelletizerMotor := TRUE;
    // Assume PelletCount is incremented by hardware feedback
END_METHOD
