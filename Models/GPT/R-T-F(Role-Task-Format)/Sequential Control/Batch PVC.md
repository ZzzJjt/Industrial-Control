VAR
    // === Phase Control ===
    Phase_Main       : INT := 0;         // 0: Idle, 1: Polymerize, 2: Decover, 3: Dry
    SubStep          : INT := 0;

    // === Process Variables ===
    Temp             : REAL;             // Reactor temperature (°C)
    Pressure         : REAL;             // Reactor pressure (bar)

    // === Setpoints and Limits ===
    TargetTemp       : REAL := 58.0;
    TempRange        : REAL := 2.0;      // ± tolerance
    PressureDropLimit: REAL := 0.8;      // bar, to detect pressure drop
    PressureInitial  : REAL := 2.0;      // bar, starting pressure

    // === Timers ===
    EvacTimer        : TON;
    HeatTimer        : TON;
    DryTimer         : TON;

    // === Timer Start Flags ===
    EvacStart        : BOOL := FALSE;
    HeatStart        : BOOL := FALSE;
    DryStart         : BOOL := FALSE;

    // === Sensor & Interlock Inputs ===
    VacuumOK         : BOOL;
    WaterCharged     : BOOL;
    SurfactantAdded  : BOOL;
    VCM_Charged      : BOOL;
    CatalystAdded    : BOOL;
    VentReady        : BOOL;
    MoistureSensorOK : BOOL;

    // === Output Control Flags ===
    ReactorEvacValve : BOOL;
    WaterValve       : BOOL;
    SurfactantPump   : BOOL;
    VCMValve         : BOOL;
    CatalystPump     : BOOL;
    HeaterOn         : BOOL;
    DryerOn          : BOOL;
    VentValve        : BOOL;
END_VAR

// === Main Phase Sequence ===
CASE Phase_Main OF

    // ----------------------------
    1: // === POLYMERIZATION PHASE ===
    CASE SubStep OF

        0: // Evacuate reactor
            EvacuateReactor();
            IF VacuumOK THEN
                SubStep := 1;
            END_IF;

        1: // Add demineralized water
            AddDemineralizedWater();
            IF WaterCharged THEN
                SubStep := 2;
            END_IF;

        2: // Add surfactants
            SurfactantPump := TRUE;
            IF SurfactantAdded THEN
                SurfactantPump := FALSE;
                SubStep := 3;
            END_IF;

        3: // Add VCM and catalyst
            VCMValve := TRUE;
            CatalystPump := TRUE;
            IF VCM_Charged AND CatalystAdded THEN
                VCMValve := FALSE;
                CatalystPump := FALSE;
                SubStep := 4;
            END_IF;

        4: // Maintain reaction temperature 55–60°C
            StartHeating(TargetTemp);
            IF Temp >= (TargetTemp - TempRange) AND Temp <= (TargetTemp + TempRange) THEN
                HeatTimer(IN := TRUE, PT := T#30m); // Hold temperature for 30 min
                HeatStart := TRUE;
                SubStep := 5;
            END_IF;

        5: // Wait for reaction and pressure drop
            IF HeatTimer.Q AND (Pressure <= PressureDropLimit) THEN
                HeatTimer(IN := FALSE);
                HeatStart := FALSE;
                HeaterOn := FALSE;
                SubStep := 0;
                Phase_Main := 2; // Transition to Decover
            END_IF;

    END_CASE;

    // ----------------------------
    2: // === DECOVER PHASE ===
    CASE SubStep OF

        0:
            IF VentReady THEN
                VentValve := TRUE;
                SubStep := 1;
            END_IF;

        1:
            // Assume depressurization takes fixed time (e.g., 5 min)
            EvacTimer(IN := TRUE, PT := T#5m);
            IF EvacTimer.Q THEN
                EvacTimer(IN := FALSE);
                VentValve := FALSE;
                SubStep := 0;
                Phase_Main := 3;
            END_IF;

    END_CASE;

    // ----------------------------
    3: // === DRY PHASE ===
    CASE SubStep OF

        0:
            DryerOn := TRUE;
            DryTimer(IN := TRUE, PT := T#20m);
            DryStart := TRUE;
            SubStep := 1;

        1:
            IF DryTimer.Q AND MoistureSensorOK THEN
                DryTimer(IN := FALSE);
                DryerOn := FALSE;
                SubStep := 0;
                Phase_Main := 0; // Batch complete
            END
