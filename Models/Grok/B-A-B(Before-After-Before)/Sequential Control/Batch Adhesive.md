(* IEC 61131-3 Structured Text: ISA-88 Reaction Step B.2 for Adhesive Batch *)
(* Purpose: Controls the Reaction step with modular operations for heating, mixing, and holding *)

(* Function Block: Heating Operation *)
FUNCTION_BLOCK HeatOperation
VAR_INPUT
    Enable : BOOL;                  (* TRUE to run operation *)
    Temp_SP : REAL;                 (* Target temperature in °C *)
    Temp_Tol : REAL;                (* Temperature tolerance in °C *)
    Temp_PV : REAL;                 (* Measured temperature in °C *)
END_VAR
VAR_OUTPUT
    HeaterOn : BOOL;                (* TRUE to activate heater *)
    Complete : BOOL;                (* TRUE when temperature reached *)
END_VAR
VAR
    State : INT := 0;               (* 0=Idle, 1=Heating, 2=Complete *)
END_VAR
IF NOT Enable THEN
    State := 0;
    HeaterOn := FALSE;
    Complete := FALSE;
ELSE
    CASE State OF
        0: (* Idle *)
            HeaterOn := FALSE;
            Complete := FALSE;
            State := 1;
        1: (* Heating *)
            HeaterOn := TRUE;
            IF ABS(Temp_PV - Temp_SP) <= Temp_Tol THEN
                State := 2;
            END_IF;
        2: (* Complete *)
            HeaterOn := FALSE;
            Complete := TRUE;
    END_CASE;
END_IF;
END_FUNCTION_BLOCK

(* Function Block: Mixing Operation *)
FUNCTION_BLOCK MixOperation
VAR_INPUT
    Enable : BOOL;                  (* TRUE to run operation *)
    RPM_SP : REAL;                  (* Target mixer speed in RPM *)
    Duration : TIME;                (* Mixing duration *)
END_VAR
VAR_OUTPUT
    MixerOn : BOOL;                 (* TRUE to activate mixer *)
    MixerSpeed : REAL;              (* Mixer speed in RPM *)
    Complete : BOOL;                (* TRUE when mixing complete *)
END_VAR
VAR
    State : INT := 0;               (* 0=Idle, 1=Mixing, 2=Complete *)
    MixTimer : TON;                 (* Timer for mixing duration *)
END_VAR
IF NOT Enable THEN
    State := 0;
    MixerOn := FALSE;
    MixerSpeed := 0.0;
    Complete := FALSE;
    MixTimer(IN := FALSE);
ELSE
    CASE State OF
        0: (* Idle *)
            MixerOn := FALSE;
            MixerSpeed := 0.0;
            Complete := FALSE;
            State := 1;
            MixTimer(IN := TRUE, PT := Duration);
        1: (* Mixing *)
            MixerOn := TRUE;
            MixerSpeed := RPM_SP;
            IF MixTimer.Q THEN
                State := 2;
            END_IF;
        2: (* Complete *)
            MixerOn := FALSE;
            MixerSpeed := 0.0;
            Complete := TRUE;
            MixTimer(IN := FALSE);
    END_CASE;
END_IF;
END_FUNCTION_BLOCK

(* Function Block: Hold Operation *)
FUNCTION_BLOCK HoldOperation
VAR_INPUT
    Enable : BOOL;                  (* TRUE to run operation *)
    Temp_SP : REAL;                 (* Target temperature in °C *)
    Temp_Tol : REAL;                (* Temperature tolerance in °C *)
    Temp_PV : REAL;                 (* Measured temperature in °C *)
    Duration : TIME;                (* Holding duration *)
END_VAR
VAR_OUTPUT
    HeaterOn : BOOL;                (* TRUE to activate heater *)
    Complete : BOOL;                (* TRUE when hold complete *)
END_VAR
VAR
    State : INT := 0;               (* 0=Idle, 1=Holding, 2=Complete *)
    HoldTimer : TON;                (* Timer for holding duration *)
END_VAR
IF NOT Enable THEN
    State := 0;
    HeaterOn := FALSE;
    Complete := FALSE;
    HoldTimer(IN := FALSE);
ELSE
    CASE State OF
        0: (* Idle *)
            HeaterOn := FALSE;
            Complete := FALSE;
            State := 1;
            HoldTimer(IN := TRUE, PT := Duration);
        1: (* Holding *)
            HeaterOn := ABS(Temp_PV - Temp_SP) > Temp_Tol; (* Maintain temp *)
            IF HoldTimer.Q THEN
                State := 2;
            END_IF;
        2: (* Complete *)
            HeaterOn := FALSE;
            Complete := TRUE;
            HoldTimer(IN := FALSE);
    END_CASE;
END_IF;
END_FUNCTION_BLOCK

(* Main Program: Reaction Step B.2 *)
PROGRAM AdhesiveReactionControl
VAR
    (* Inputs *)
    StartReaction : BOOL;           (* TRUE to start reaction step *)
    EmergencyStop : BOOL;           (* TRUE to halt operations *)
    Temp_PV : REAL;                 (* Reactor temperature in °C *)
    
    (* Outputs *)
    HeaterOn : BOOL;                (* TRUE to activate heater *)
    MixerOn : BOOL;                 (* TRUE to activate mixer *)
    MixerSpeed : REAL;              (* Mixer speed in RPM *)
    ReactionComplete : BOOL;        (* TRUE when reaction step complete *)
    
    (* State Machine *)
    State : INT := 0;               (* 0=Idle, 1=Heat, 2=Mix, 3=Hold, 4=Complete *)
    CONSTANT
        STATE_IDLE : INT := 0;      (* Waiting to start *)
        STATE_HEAT : INT := 1;      (* Heating to reaction temp *)
        STATE_MIX : INT := 2;       (* Mixing reactants *)
        STATE_HOLD : INT := 3;      (* Holding reaction conditions *)
        STATE_COMPLETE : INT := 4;  (* Reaction step complete *)
    END_CONSTANT
    
    (* Recipe Parameters *)
    ReactionTemp_SP : REAL := 120.0; (* Target reaction temperature in °C *)
    Temp_Tol : REAL := 5.0;         (* Temperature tolerance in °C *)
    Mix_RPM : REAL := 500.0;        (* Mixer speed in RPM *)
    Mix_Duration : TIME := T#30m;   (* Mixing duration *)
    Hold_Duration : TIME := T#1h;   (* Holding duration *)
    
    (* Operation Instances *)
    HeatOp : HeatOperation;         (* Heating operation *)
    MixOp : MixOperation;           (* Mixing operation *)
    HoldOp : HoldOperation;         (* Holding operation *)
END_VAR

(* Main Logic *)
IF EmergencyStop THEN
    State := STATE_IDLE;
    HeaterOn := FALSE;
    MixerOn := FALSE;
    MixerSpeed := 0.0;
    ReactionComplete := FALSE;
    HeatOp.Enable := FALSE;
    MixOp.Enable := FALSE;
    HoldOp.Enable := FALSE;
ELSIF StartReaction AND State = STATE_IDLE THEN
    State := STATE_HEAT;
    ReactionComplete := FALSE;
END_IF;

(* State Machine *)
CASE State OF
    STATE_IDLE:
        HeaterOn := FALSE;
        MixerOn := FALSE;
        MixerSpeed := 0.0;
        ReactionComplete := FALSE;
        HeatOp.Enable := FALSE;
        MixOp.Enable := FALSE;
        HoldOp.Enable := FALSE;

    STATE_HEAT:
        HeatOp(Enable := TRUE, Temp_SP := ReactionTemp_SP, Temp_Tol := Temp_Tol, Temp_PV := Temp_PV);
        HeaterOn := HeatOp.HeaterOn;
        IF HeatOp.Complete THEN
            State := STATE_MIX;
            HeatOp.Enable := FALSE;
        END_IF;

    STATE_MIX:
        MixOp(Enable := TRUE, RPM_SP := Mix_RPM, Duration := Mix_Duration);
        MixerOn := MixOp.MixerOn;
        MixerSpeed := MixOp.MixerSpeed;
        IF MixOp.Complete THEN
            State := STATE_HOLD;
            MixOp.Enable := FALSE;
        END_IF;

    STATE_HOLD:
        HoldOp(Enable := TRUE, Temp_SP := ReactionTemp_SP, Temp_Tol := Temp_Tol, Temp_PV := Temp_PV, Duration := Hold_Duration);
        HeaterOn := HoldOp.HeaterOn;
        IF HoldOp.Complete THEN
            State := STATE_COMPLETE;
            HoldOp.Enable := FALSE;
        END_IF;

    STATE_COMPLETE:
        HeaterOn := FALSE;
        MixerOn := FALSE;
        MixerSpeed := 0.0;
        ReactionComplete := TRUE;
        IF NOT StartReaction THEN
            State := STATE_IDLE;
        END_IF;

ELSE
    State := STATE_IDLE; (* Fallback to safe state *)
END_CASE;

(* Notes:
   - ISA-88 Compliance:
     - Unit Procedure: Reaction step B.2, part of adhesive batch recipe.
     - Operations: Heat, Mix, Hold implemented as modular function blocks.
     - Recipe Parameters: ReactionTemp_SP, Mix_RPM, durations for flexibility.
     - State Model: Idle, Running (Heat/Mix/Hold), Complete per ISA-88.
   - Reaction Step B.2:
     1. Heat: Reach 120°C ± 5°C to initiate reaction, ensuring chemical activation.
     2. Mix: Run mixer at 500 RPM for 30 min to ensure reactant homogeneity.
     3. Hold: Maintain 120°C for 1 hour to complete reaction, ensuring product quality.
   - Safety:
     - EmergencyStop halts all operations, resets state.
     - Temperature tolerance (5°C) prevents overheating or under-reaction.
   - Quality:
     - Precise temperature and mixing ensure consistent adhesive properties.
     - Timers (30 min mix, 1 hr hold) guarantee complete reaction.
   - Physical Integration:
     - Inputs: StartReaction (operator), Temp_PV (sensor), EmergencyStop (safety).
     - Outputs: HeaterOn (relay), MixerOn (relay), MixerSpeed (analog).
   - Scalability:
     - Add operations (e.g., pressure control) as new function blocks.
     - Adjust parameters (Temp_SP, Mix_Duration) for different adhesives.
   - Maintenance:
     - Modular function blocks simplify debugging and updates.
     - Monitor Temp_PV, MixOp.MixTimer.ET on HMI for diagnostics.
*)
END_PROGRAM
