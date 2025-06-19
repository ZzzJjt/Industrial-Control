(* ISA-88-Compliant Reaction Stage for Urea Fertilizer Production *)
PROGRAM UreaReactionStage
VAR
    Step            : INT := 0; // Batch phase step
    Timer           : TON;      // General-purpose phase timer
    TimerEnable     : BOOL;
    TimerPreset     : TIME;
    ReactionComplete : BOOL := FALSE;

    // Process Parameters
    TargetTemp      : REAL := 180.0;  // °C
    TargetPressure  : REAL := 140.0;  // bar
    ReactionTime    : TIME := T#30m;

    // Inputs
    Temp_PV         : REAL;          // Measured temperature
    Pressure_PV     : REAL;          // Measured pressure

    // Outputs
    HeaterCmd       : BOOL;
    PressureValveCmd: BOOL;
    CoolerCmd       : BOOL;
END_VAR

// Timer call
Timer(IN := TimerEnable, PT := TimerPreset);

CASE Step OF

    0: // Start Heating Phase
        HeaterCmd := TRUE;
        CoolerCmd := FALSE;
        PressureValveCmd := FALSE;
        IF Temp_PV >= TargetTemp THEN
            TimerEnable := TRUE;
            TimerPreset := T#10s; // Short stabilization delay
            Step := 1;
        END_IF

    1: // Heating Stabilization Timer
        IF Timer.Q THEN
            TimerEnable := FALSE;
            Step := 2;
        END_IF

    2: // Pressure Regulation Phase
        HeaterCmd := FALSE;
        PressureValveCmd := TRUE;
        IF Pressure_PV >= TargetPressure THEN
            TimerEnable := TRUE;
            TimerPreset := ReactionTime;
            Step := 3;
        END_IF

    3: // Hold Reaction Conditions
        IF Timer.Q THEN
            TimerEnable := FALSE;
            Step := 4;
        END_IF

    4: // Cooling Phase
        PressureValveCmd := FALSE;
        CoolerCmd := TRUE;
        IF Temp_PV <= 60.0 THEN // Assume safe cooling temperature
            Step := 5;
        END_IF

    5: // Reaction Complete
        CoolerCmd := FALSE;
        ReactionComplete := TRUE;
        // Ready for next stage
END_CASE

// Modular Callouts (illustrative comments)
// StartHeating: HeaterCmd := TRUE until Temp_PV >= TargetTemp
// RegulatePressure: PressureValveCmd := TRUE until Pressure_PV >= TargetPressure
// StartCooling: CoolerCmd := TRUE until Temp_PV <= CoolingThreshold

(* Notes:
   - Structured Text (ST) supports ISA-88 modularity via CASE and encapsulated methods
   - Timers and boolean flags simulate phases within PLC scan cycle model
   - Synchronizing temp and pressure requires tight process monitoring and edge logic
*)
