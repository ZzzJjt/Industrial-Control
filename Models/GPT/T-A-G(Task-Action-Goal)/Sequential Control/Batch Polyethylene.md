// IEC 61131-3 Structured Text Program
// ISA-88-Compliant Batch Control for Polyethylene Production

VAR
    Phase : INT := 0; // Batch step tracker
    StepTimer : TON;
    TimerTrigger : BOOL;
    TimerDone : BOOL;

    RawMaterials_Ready : BOOL := FALSE;
    Reaction_Complete : BOOL := FALSE;
    Quench_OK : BOOL := FALSE;
    Dry_OK : BOOL := FALSE;
    Pellet_OK : BOOL := FALSE;
    QC_OK : BOOL := FALSE;
    Packaged : BOOL := FALSE;

    Temp_PV : REAL;
    Temp_SP : REAL := 80.0;
    Temp_Tolerance : REAL := 2.0;

    Pressure_PV : REAL;
    Pressure_SP : REAL := 15.0;
    Pressure_Tolerance : REAL := 1.0;

    Batch_Complete : BOOL := FALSE;
END_VAR

// === Main Batch Sequence ===
CASE Phase OF

    0: // Raw Material Preparation
        AddMaterial('Ethylene', 1000.0);
        AddMaterial('Catalyst', 50.0);
        RawMaterials_Ready := TRUE;
        IF RawMaterials_Ready THEN
            Phase := 1;
        END_IF

    1: // Polymerization
        StartReaction();
        IF WithinRange(Temp_PV, Temp_SP, Temp_Tolerance) AND 
           WithinRange(Pressure_PV, Pressure_SP, Pressure_Tolerance) THEN
            StartTimer(T#30m);
            Phase := 2;
        END_IF

    2: // Reaction Duration Countdown
        IF TimerDone THEN
            Reaction_Complete := TRUE;
            Phase := 3;
        END_IF

    3: // Quenching
        QuenchReactor();
        Quench_OK := TRUE;
        Phase := 4;

    4: // Drying
        BeginDrying();
        StartTimer(T#15m);
        IF TimerDone THEN
            Dry_OK := TRUE;
            Phase := 5;
        END_IF

    5: // Pelletizing
        Pelletize();
        Pellet_OK := TRUE;
        Phase := 6;

    6: // Quality Control
        PerformQC();
        IF QC_OK THEN
            Phase := 7;
        END_IF

    7: // Packaging and Storage
        PackageProduct();
        StoreProduct();
        Packaged := TRUE;
        Batch_Complete := TRUE;

END_CASE

// === Function Calls ===

FUNCTION AddMaterial
VAR_INPUT MaterialName : STRING; Volume : REAL; END_VAR
// Simulated add-material routine with internal confirmation

FUNCTION StartReaction
// Start reactor heating, pressure control, agitation

FUNCTION QuenchReactor
// Inject cooling media to halt reaction safely

FUNCTION BeginDrying
// Start dryer fan and heat zones

FUNCTION Pelletize
// Control extruder and pelletizing blade

FUNCTION PerformQC
// Simulate online QC logic (e.g., density, melting point)

FUNCTION PackageProduct
// Wrap and seal into bags or drums

FUNCTION StoreProduct
// Place in warehouse buffer

FUNCTION StartTimer
VAR_INPUT Duration : TIME; END_VAR
TimerTrigger := TRUE;
StepTimer(IN := TimerTrigger, PT := Duration);
TimerDone := StepTimer.Q;

FUNCTION WithinRange : BOOL
VAR_INPUT PV : REAL; SP : REAL; Tol : REAL; END_VAR
WithinRange := ABS(PV - SP) <= Tol;
