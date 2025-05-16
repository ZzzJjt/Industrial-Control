(* IEC 61131-3 Structured Text: 3D Pouch Making Machine Control *)
(* Purpose: Manages start-up and shutdown sequences with tension regulation *)

PROGRAM PouchMachineControl
VAR
    (* Inputs *)
    StartCmd : BOOL;                (* TRUE to initiate start-up *)
    StopCmd : BOOL;                 (* TRUE to initiate shutdown *)
    EmergencyStop : BOOL;           (* TRUE to halt all operations *)
    HeaterTemp_PV : REAL;           (* Measured heater temperature in °C *)
    CoolerTemp_PV : REAL;           (* Measured cooler temperature in °C *)
    Tension_PV : REAL;              (* Measured winding tension in N *)
    MaterialFlow_PV : REAL;         (* Measured material flow rate in m/s *)

    (* Outputs *)
    HeaterOn : BOOL;                (* TRUE to activate heaters *)
    CoolerOn : BOOL;                (* TRUE to activate cooling stations *)
    FeederOn : BOOL;                (* TRUE to activate feeders *)
    FeederSpeed : REAL;             (* Feeder speed in % (0.0 to 100.0) *)
    HorizontalCutterOn : BOOL;      (* TRUE to activate horizontal cutter *)
    VerticalCutterOn : BOOL;        (* TRUE to activate vertical cutter *)
    MachineRunning : BOOL;          (* TRUE when machine is fully operational *)
    ShutdownComplete : BOOL;        (* TRUE when shutdown is complete *)

    (* State Machine *)
    State : INT := 0;               (* Current state: 0=Idle, 1=Preheat, ..., 7=Cooldown *)
    CONSTANT
        STATE_IDLE : INT := 0;      (* Idle, waiting for start *)
        STATE_PREHEAT : INT := 1;   (* Preheating heaters *)
        STATE_COOLER_ACTIVATE : INT := 2; (* Activating cooling stations *)
        STATE_FEEDER_START : INT := 3; (* Starting feeders, regulating tension *)
        STATE_CUTTER_ACTIVATE : INT := 4; (* Activating cutters *)
        STATE_RUNNING : INT := 5;   (* Full operation *)
        STATE_SHUTDOWN_CUTTERS : INT := 6; (* Stopping cutters, releasing tension *)
        STATE_COOLDOWN : INT := 7;  (* Cooling down heaters *)
        STATE_SHUTDOWN_COMPLETE : INT := 8; (* Shutdown complete *)
    END_CONSTANT

    (* Parameters *)
    HeaterTemp_SP : REAL := 180.0;  (* Target heater temperature in °C *)
    HeaterTemp_Tol : REAL := 5.0;   (* Temperature tolerance in °C *)
    CoolerTemp_SP : REAL := 20.0;   (* Target cooler temperature in °C *)
    CoolerTemp_Tol : REAL := 3.0;   (* Cooler temperature tolerance *)
    Tension_SP : REAL := 50.0;      (* Target winding tension in N *)
    Tension_Tol : REAL := 5.0;      (* Tension tolerance in N *)
    Flow_SP : REAL := 0.5;          (* Target material flow rate in m/s *)
    Flow_Tol : REAL := 0.05;        (* Flow tolerance in m/s *)
    SafeHeaterTemp : REAL := 50.0;  (* Safe temperature for shutdown *)

    (* Timing *)
    PreheatTimer : TON;             (* Timer for preheating *)
    CooldownTimer : TON;            (* Timer for cooldown *)
    TensionStabilizeTimer : TON;    (* Timer for tension stabilization *)
    Preheat_Duration : TIME := T#5m; (* Preheating time *)
    Cooldown_Duration : TIME := T#10m; (* Cooldown time *)
    TensionStabilize_Duration : TIME := T#10s; (* Tension stabilization time *)

    (* Tension Control *)
    TensionError : REAL;            (* Tension error (Tension_SP - Tension_PV) *)
    Kp_Tension : REAL := 0.5;       (* Proportional gain for tension control *)
END_VAR

(* Method: Regulate Winding Tension *)
METHOD PRIVATE RegulateTension : BOOL
TensionError := Tension_SP - Tension_PV;
FeederSpeed := FeederSpeed + (Kp_Tension * TensionError); (* Simple P-control *)
IF FeederSpeed > 100.0 THEN
    FeederSpeed := 100.0;       (* Clamp to max *)
ELSIF FeederSpeed < 0.0 THEN
    FeederSpeed := 0.0;         (* Clamp to min *)
END_IF;
RegulateTension := (ABS(TensionError) <= Tension_Tol); (* TRUE if within tolerance *)
END_METHOD

(* Method: Check Conditions *)
METHOD PRIVATE CheckConditions : BOOL
VAR_INPUT
    Temp : REAL;                (* Measured temperature *)
    Temp_SP : REAL;             (* Target temperature *)
    Temp_Tol : REAL;            (* Tolerance *)
    OtherCondition : BOOL;      (* Additional condition, e.g., flow *)
END_VAR
CheckConditions := (ABS(Temp - Temp_SP) <= Temp_Tol) AND OtherCondition;
END_METHOD

(* Main Cyclic Logic *)
IF EmergencyStop THEN
    State := STATE_IDLE;
    HeaterOn := FALSE;
    CoolerOn := FALSE;
    FeederOn := FALSE;
    FeederSpeed := 0.0;
    HorizontalCutterOn := FALSE;
    VerticalCutterOn := FALSE;
    MachineRunning := FALSE;
    ShutdownComplete := FALSE;
    PreheatTimer(IN := FALSE);
    CooldownTimer(IN := FALSE);
    TensionStabilizeTimer(IN := FALSE);
ELSIF StopCmd AND State <> STATE_IDLE THEN
    State := STATE_SHUTDOWN_CUTTERS; (* Initiate shutdown *)
END_IF;

IF StartCmd AND State = STATE_IDLE THEN
    State := STATE_PREHEAT;         (* Start sequence *)
    ShutdownComplete := FALSE;
    MachineRunning := FALSE;
END_IF;

(* State Machine *)
CASE State OF
    STATE_IDLE:
        HeaterOn := FALSE;
        CoolerOn := FALSE;
        FeederOn := FALSE;
        FeederSpeed := 0.0;
        HorizontalCutterOn := FALSE;
        VerticalCutterOn := FALSE;
        MachineRunning := FALSE;
        PreheatTimer(IN := FALSE);
        CooldownTimer(IN := FALSE);
        TensionStabilizeTimer(IN := FALSE);

    STATE_PREHEAT:
        HeaterOn := TRUE;
        PreheatTimer(IN := TRUE, PT := Preheat_Duration);
        IF PreheatTimer.Q AND CheckConditions(HeaterTemp_PV, HeaterTemp_SP, HeaterTemp_Tol, TRUE) THEN
            State := STATE_COOLER_ACTIVATE;
            PreheatTimer(IN := FALSE);
        END_IF;

    STATE_COOLER_ACTIVATE:
        CoolerOn := TRUE;
        IF CheckConditions(CoolerTemp_PV, CoolerTemp_SP, CoolerTemp_Tol, TRUE) THEN
            State := STATE_FEEDER_START;
        END_IF;

    STATE_FEEDER_START:
        FeederOn := TRUE;
        TensionStabilizeTimer(IN := TRUE, PT := TensionStabilize_Duration);
        IF RegulateTension() AND TensionStabilizeTimer.Q AND CheckConditions(MaterialFlow_PV, Flow_SP, Flow_Tol, TRUE) THEN
            State := STATE_CUTTER_ACTIVATE;
            TensionStabilizeTimer(IN := FALSE);
        END_IF;

    STATE_CUTTER_ACTIVATE:
        HorizontalCutterOn := TRUE;
        VerticalCutterOn := TRUE;
        IF RegulateTension() AND CheckConditions(MaterialFlow_PV, Flow_SP, Flow_Tol, TRUE) THEN
            State := STATE_RUNNING;
            MachineRunning := TRUE;
        END_IF;

    STATE_RUNNING:
        IF NOT RegulateTension() OR NOT CheckConditions(MaterialFlow_PV, Flow_SP, Flow_Tol, TRUE) THEN
            State := STATE_SHUTDOWN_CUTTERS; (* Fault: tension or flow out of range *)
        END_IF;

    STATE_SHUTDOWN_CUTTERS:
        HorizontalCutterOn := FALSE;
        VerticalCutterOn := FALSE;
        FeederSpeed := FeederSpeed * 0.9; (* Gradually reduce speed *)
        IF FeederSpeed < 1.0 THEN
            FeederOn := FALSE;
            FeederSpeed := 0.0;
            State := STATE_COOLDOWN;
        END_IF;

    STATE_COOLDOWN:
        HeaterOn := FALSE;
        CooldownTimer(IN := TRUE, PT := Cooldown_Duration);
        IF CooldownTimer.Q AND HeaterTemp_PV <= SafeHeaterTemp THEN
            CoolerOn := FALSE;
            State := STATE_SHUTDOWN_COMPLETE;
            CooldownTimer(IN := FALSE);
        END_IF;

    STATE_SHUTDOWN_COMPLETE:
        ShutdownComplete := TRUE;
        MachineRunning := FALSE;
        State := STATE_IDLE;

ELSE
    State := STATE_IDLE; (* Fallback to safe state *)
END_CASE;

(* Notes:
   - Start-Up Sequence:
     1. Preheat heaters to 180°C ± 5°C (5 min) to ensure material formability.
     2. Activate coolers to 20°C ± 3°C for pouch stabilization.
     3. Start feeders, regulate tension to 50 N ± 5 N to prevent stretching/wrinkling.
     4. Activate cutters once flow (0.5 m/s ± 0.05 m/s) and tension are stable.
   - Shutdown Sequence:
     1. Stop cutters to prevent material damage.
     2. Gradually reduce feeder speed to release tension, avoiding material snap.
     3. Cool heaters to <50°C (10 min) for safety.
     4. Deactivate coolers, complete shutdown.
   - Winding Tension:
     - Critical for pouch quality: prevents stretching (over-tension) or wrinkling (under-tension).
     - Regulated via P-control (Kp=0.5) adjusting FeederSpeed based on Tension_PV.
   - Safety:
     - EmergencyStop halts all components, resets state.
     - Interlocks ensure temperature, tension, and flow are within tolerances before advancing.
   - Efficiency:
     - Timers (PreheatTimer, CooldownTimer) ensure sufficient preparation and safe shutdown.
     - State machine minimizes unnecessary operations, optimizing cycle time.
   - Physical Integration:
     - Inputs: StartCmd/StopCmd (operator buttons), HeaterTemp_PV/Tension_PV (sensors).
     - Outputs: HeaterOn/CoolerOn (relays), FeederSpeed (analog), CutterOn (actuators).
   - Scalability:
     - Add states for additional stations (e.g., sealing).
     - Adjust setpoints (HeaterTemp_SP, Tension_SP) for different materials.
   - Maintenance:
     - Monitor TensionError, FeederSpeed, and timers on HMI for diagnostics.
     - Log state transitions for process audits.
*)
END_PROGRAM
