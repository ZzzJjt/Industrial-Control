(* Valmet Paper Machine - Press Section Startup Control *)
PROGRAM PressSectionStartup

VAR
    (* Phase Control *)
    StartupPhase : INT := 0;
    Timer : TON;
    TimerStart : BOOL;

    (* Roll & Conveyor Control *)
    PressRollSpeed : REAL := 0.0; (* m/min *)
    TargetSpeed : REAL := 500.0;
    SpeedRampRate : REAL := 10.0; (* m/min per second *)

    NipPressure : REAL := 0.0; (* kN/m *)
    TargetPressure : REAL := 250.0;
    PressureRampRate : REAL := 5.0; (* kN/m per second *)

    Temperature : REAL := 20.0;
    TargetTemperature : REAL := 85.0;
    HeatingEnabled : BOOL := FALSE;

    ConveyorRunning : BOOL := FALSE;
    RollsEnabled : BOOL := FALSE;

    (* Safety & Interlocks *)
    SafetyOK : BOOL;
    TempSensorOK : BOOL;
    PressureSensorOK : BOOL;
    AllInterlocksOK : BOOL;
END_VAR

(* Check overall interlock status *)
AllInterlocksOK := SafetyOK AND TempSensorOK AND PressureSensorOK;

(* Startup Sequence *)
CASE StartupPhase OF

    0: (* Initialization & Safety Check *)
        IF AllInterlocksOK THEN
            StartupPhase := 1;
        END_IF

    1: (* Enable Rolls & Conveyor *)
        RollsEnabled := TRUE;
        ConveyorRunning := TRUE;
        TimerStart := TRUE;
        Timer(IN := TimerStart, PT := T#5s);
        IF Timer.Q THEN
            TimerStart := FALSE;
            StartupPhase := 2;
        END_IF

    2: (* Ramp-Up Speed *)
        IF PressRollSpeed < TargetSpeed THEN
            PressRollSpeed := PressRollSpeed + (SpeedRampRate * 0.1);
        ELSE
            PressRollSpeed := TargetSpeed;
            StartupPhase := 3;
        END_IF

    3: (* Ramp-Up Pressure *)
        IF NipPressure < TargetPressure THEN
            NipPressure := NipPressure + (PressureRampRate * 0.1);
        ELSE
            NipPressure := TargetPressure;
            StartupPhase := 4;
        END_IF

    4: (* Heat-Up *)
        HeatingEnabled := TRUE;
        IF Temperature < TargetTemperature THEN
            Temperature := Temperature + 0.5; (* Simplified temperature increase *)
        ELSE
            Temperature := TargetTemperature;
            StartupPhase := 5;
        END_IF

    5: (* Startup Complete *)
        (* System is fully started and running *)
        (* Optionally signal to operator or higher-level logic *)
END_CASE

(* Notes:
 - Maintaining proper nip pressure prevents crushing or uneven dewatering of the paper web.
 - Stable temperature ensures consistent moisture removal and web smoothness.
 - Synchronization avoids wrinkling or tearing during acceleration. *)

END_PROGRAM
