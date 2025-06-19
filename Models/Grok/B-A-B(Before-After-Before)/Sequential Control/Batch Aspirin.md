(* IEC 61131-3 Structured Text: ISA-88 Aspirin Batch Control *)
(* Purpose: Controls Reaction, Crystallization, and Drying stages *)

(* Function Block: Heat Operation *)
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

(* Function Block: Maintain Operation *)
FUNCTION_BLOCK MaintainOperation
VAR_INPUT
    Enable : BOOL;                  (* TRUE to run operation *)
    Temp_SP : REAL;                 (* Target temperature in °C *)
    Temp_Tol : REAL;                (* Temperature tolerance in °C *)
    Temp_PV : REAL;                 (* Measured temperature in °C *)
    Duration : TIME;                (* Duration to maintain conditions *)
    AgitatorOn : BOOL;              (* TRUE to enable agitator *)
END_VAR
VAR_OUTPUT
    HeaterOn : BOOL;                (* TRUE to activate heater *)
    AgitatorActive : BOOL;          (* TRUE when agitator is running *)
    Complete : BOOL;                (* TRUE when duration complete *)
END_VAR
VAR
    State : INT := 0;               (* 0=Idle, 1=Maintaining, 2=Complete *)
    Timer : TON;                    (* Timer for duration *)
END_VAR
IF NOT Enable THEN
    State := 0;
    HeaterOn := FALSE;
    AgitatorActive := FALSE;
    Complete := FALSE;
    Timer(IN := FALSE);
ELSE
    CASE State OF
        0: (* Idle *)
            HeaterOn := FALSE;
            AgitatorActive := FALSE;
            Complete := FALSE;
            State := 1;
            Timer(IN := TRUE, PT := Duration);
        1: (* Maintaining *)
            HeaterOn := ABS(Temp_PV - Temp_SP) > Temp_Tol;
            AgitatorActive := AgitatorOn;
            IF Timer.Q THEN
                State := 2;
            END_IF;
        2: (* Complete *)
            HeaterOn := FALSE;
            AgitatorActive := FALSE;
            Complete := TRUE;
            Timer(IN := FALSE);
    END_CASE;
END_IF;
END_FUNCTION_BLOCK

(* Function Block: Cool Operation *)
FUNCTION_BLOCK CoolOperation
VAR_INPUT
    Enable : BOOL;                  (* TRUE to run operation *)
    Temp_SP : REAL;                 (* Target temperature in °C *)
    Temp_Tol : REAL;                (* Temperature tolerance in °C *)
    Temp_PV : REAL;                 (* Measured temperature in °C *)
END_VAR
VAR_OUTPUT
    CoolerOn : BOOL;                (* TRUE to activate cooler *)
    Complete : BOOL;                (* TRUE when temperature reached *)
END_VAR
VAR
    State : INT := 0;               (* 0=Idle, 1=Cooling, 2=Complete *)
END_VAR
IF NOT Enable THEN
    State := 0;
    CoolerOn := FALSE;
    Complete := FALSE;
ELSE
    CASE State OF
        0: (* Idle *)
            CoolerOn := FALSE;
            Complete := FALSE;
            State := 1;
        1: (* Cooling *)
            CoolerOn := TRUE;
            IF ABS(Temp_PV - Temp_SP) <= Temp_Tol THEN
                State := 2;
            END_IF;
        2: (* Complete *)
            CoolerOn := FALSE;
            Complete := TRUE;
    END_CASE;
END_IF;
END_FUNCTION_BLOCK

(* Main Program: Aspirin Batch Control *)
PROGRAM AspirinBatchControl
VAR
    (* Inputs *)
    StartBatch : BOOL;              (* TRUE to start batch *)
    EmergencyStop : BOOL;           (* TRUE to halt operations *)
    ReactorTemp_PV : REAL;          (* Reactor temperature in °C *)
    CrystTemp_PV : REAL;            (* Crystallizer temperature in °C *)
    DryerTemp_PV : REAL;            (* Dryer temperature in °C *)
    
    (* Outputs *)
    ReactorHeaterOn : BOOL;         (* TRUE to activate reactor heater *)
    ReactorAgitatorOn : BOOL;       (* TRUE to activate reactor agitator *)
    CrystCoolerOn : BOOL;           (* TRUE to activate crystallizer cooler *)
    DryerHeaterOn : BOOL;           (* TRUE to activate dryer heater *)
    StageComplete : BOOL;           (* TRUE when current stage complete *)
    
    (* State Machine *)
    State : INT := 0;               (* 0=Idle, 1=Reaction, 2=Crystallization, 3=Drying, 4=Complete *)
    CONSTANT
        STATE_IDLE : INT := 0;      (* Waiting to start *)
        STATE_REACTION : INT := 1;  (* Reaction stage *)
        STATE_CRYSTALLIZATION : INT := 2; (* Crystallization stage *)
        STATE_DRYING : INT := 3;    (* Drying stage *)
        STATE_COMPLETE : INT := 4;  (* Batch stage complete *)
    END_CONSTANT
    
    (* Sub-State for Reaction *)
    ReactionSubState : INT := 0;    (* 0=Idle, 1=Heat, 2=Maintain, 3=Complete *)
    CONSTANT
        REACT_IDLE : INT := 0;
        REACT_HEAT : INT := 1;
        REACT_MAINTAIN : INT := 2;
        REACT_COMPLETE : INT := 3;
    END_CONSTANT
    
    (* Sub-State for Crystallization *)
    CrystSubState : INT := 0;       (* 0=Idle, 1=Cool, 2=Hold, 3=Complete *)
    CONSTANT
        CRYST_IDLE : INT := 0;
        CRYST_COOL : INT := 1;
        CRYST_HOLD : INT := 2;
        CRYST_COMPLETE : INT := 3;
    END_CONSTANT
    
    (* Recipe Parameters *)
    ReactionTemp_SP : REAL := 80.0; (* Reaction temperature in °C *)
    ReactionTemp_Tol : REAL := 3.0; (* Temperature tolerance in °C *)
    ReactionDuration : TIME := T#2h; (* Reaction duration *)
    CrystTemp_SP : REAL := 20.0;    (* Crystallization temperature in °C *)
    CrystTemp_Tol : REAL := 2.0;    (* Crystallization tolerance in °C *)
    CrystHoldDuration : TIME := T#1h; (* Crystallization hold duration *)
    DryerTemp_SP : REAL := 90.0;    (* Drying temperature in °C *)
    DryerTemp_Tol : REAL := 3.0;    (* Drying tolerance in °C *)
    DryerDuration : TIME := T#4h;   (* Drying duration *)
    
    (* Operation Instances *)
    HeatOp : HeatOperation;         (* Heating operation *)
    MaintainOp : MaintainOperation; (* Maintain operation *)
    CoolOp : CoolOperation;         (* Cooling operation *)
END_VAR

(* Main Logic *)
IF EmergencyStop THEN
    State := STATE_IDLE;
    ReactionSubState := REACT_IDLE;
    CrystSubState := CRYST_IDLE;
    ReactorHeaterOn := FALSE;
    ReactorAgitatorOn := FALSE;
    CrystCoolerOn := FALSE;
    DryerHeaterOn := FALSE;
    StageComplete := FALSE;
    HeatOp.Enable := FALSE;
    MaintainOp.Enable := FALSE;
    CoolOp.Enable := FALSE;
ELSIF StartBatch AND State = STATE_IDLE THEN
    State := STATE_REACTION;
    ReactionSubState := REACT_HEAT;
    StageComplete := FALSE;
END_IF;

(* State Machine *)
CASE State OF
    STATE_IDLE:
        ReactorHeaterOn := FALSE;
        ReactorAgitatorOn := FALSE;
        CrystCoolerOn := FALSE;
        DryerHeaterOn := FALSE;
        StageComplete := FALSE;
        ReactionSubState := REACT_IDLE;
        CrystSubState := CRYST_IDLE;
        HeatOp.Enable := FALSE;
        MaintainOp.Enable := FALSE;
        CoolOp.Enable := FALSE;

    STATE_REACTION:
        CASE ReactionSubState OF
            REACT_IDLE:
                ReactorHeaterOn := FALSE;
                ReactorAgitatorOn := FALSE;
                ReactionSubState := REACT_HEAT;
            REACT_HEAT:
                HeatOp(Enable := TRUE, Temp_SP := ReactionTemp_SP, Temp_Tol := ReactionTemp_Tol, Temp_PV := ReactorTemp_PV);
                ReactorHeaterOn := HeatOp.HeaterOn;
                IF HeatOp.Complete THEN
                    ReactionSubState := REACT_MAINTAIN;
                    HeatOp.Enable := FALSE;
                END_IF;
            REACT_MAINTAIN:
                MaintainOp(Enable := TRUE, Temp_SP := ReactionTemp_SP, Temp_Tol := ReactionTemp_Tol, Temp_PV := ReactorTemp_PV, Duration := ReactionDuration, AgitatorOn := TRUE);
                ReactorHeaterOn := MaintainOp.HeaterOn;
                ReactorAgitatorOn := MaintainOp.AgitatorActive;
                IF MaintainOp.Complete THEN
                    ReactionSubState := REACT_COMPLETE;
                    MaintainOp.Enable := FALSE;
                END_IF;
            REACT_COMPLETE:
                ReactorHeaterOn := FALSE;
                ReactorAgitatorOn := FALSE;
                State := STATE_CRYSTALLIZATION;
                CrystSubState := CRYST_COOL;
        END_CASE;

    STATE_CRYSTALLIZATION:
        CASE CrystSubState OF
            CRYST_IDLE:
                CrystCoolerOn := FALSE;
                CrystSubState := CRYST_COOL;
            CRYST_COOL:
                CoolOp(Enable := TRUE, Temp_SP := CrystTemp_SP, Temp_Tol := CrystTemp_Tol, Temp_PV := CrystTemp_PV);
                CrystCoolerOn := CoolOp.CoolerOn;
                IF CoolOp.Complete THEN
                    CrystSubState := CRYST_HOLD;
                    CoolOp.Enable := FALSE;
                END_IF;
            CRYST_HOLD:
                MaintainOp(Enable := TRUE, Temp_SP := CrystTemp_SP, Temp_Tol := CrystTemp_Tol, Temp_PV := CrystTemp_PV, Duration := CrystHoldDuration, AgitatorOn := FALSE);
                CrystCoolerOn := MaintainOp.HeaterOn; (* Repurposed for cooling control *)
                IF MaintainOp.Complete THEN
                    CrystSubState := CRYST_COMPLETE;
                    MaintainOp.Enable := FALSE;
                END_IF;
            CRYST_COMPLETE:
                CrystCoolerOn := FALSE;
                State := STATE_DRYING;
        END_CASE;

    STATE_DRYING:
        HeatOp(Enable := TRUE, Temp_SP := DryerTemp_SP, Temp_Tol := DryerTemp_Tol, Temp_PV := DryerTemp_PV);
        DryerHeaterOn := HeatOp.HeaterOn;
        IF HeatOp.Complete THEN
            MaintainOp(Enable := TRUE, Temp_SP := DryerTemp_SP, Temp_Tol := DryerTemp_Tol, Temp_PV := DryerTemp_PV, Duration := DryerDuration, AgitatorOn := FALSE);
            DryerHeaterOn := MaintainOp.HeaterOn;
            IF MaintainOp.Complete THEN
                State := STATE_COMPLETE;
                MaintainOp.Enable := FALSE;
            END_IF;
        END_IF;

    STATE_COMPLETE:
        ReactorHeaterOn := FALSE;
        ReactorAgitatorOn := FALSE;
        CrystCoolerOn := FALSE;
        DryerHeaterOn := FALSE;
        StageComplete := TRUE;
        IF NOT StartBatch THEN
            State := STATE_IDLE;
        END_IF;

ELSE
    State := STATE_IDLE; (* Fallback to safe state *)
END_CASE;

(* Notes:
   - ISA-88 Compliance:
     - Unit Procedures: Reaction, Crystallization, Drying, each with modular operations.
     - Operations: Heat, Maintain, Cool implemented as reusable function blocks.
     - Recipe Parameters: Temp_SP, Duration, etc., for flexibility across recipes.
     - State Model: Idle, Running (Heat/Maintain/Cool), Complete per ISA-88.
   - Process Stages:
     - Reaction: Heat to 80°C ± 3°C, maintain with agitation for 2 hours to form acetylsalicylic acid.
     - Crystallization: Cool to 20°C ± 2°C, hold for 1 hour to form aspirin crystals.
     - Drying: Heat to 90°C ± 3°C for 4 hours to remove moisture, ensuring pure aspirin.
   - Inputs: Salicylic acid, acetic anhydride, sulfuric acid (loaded in A.1, not controlled here).
   - Outputs: Acetylsalicylic acid (crystals), acetic acid (byproduct removed in D.2).
   - Safety:
     - EmergencyStop halts all operations, resets state.
     - Temperature tolerances (3°C, 2°C) prevent quality issues.
   - Quality:
     - Precise temperature and timing ensure consistent reaction, crystal size, and dryness.
     - Agitation in reaction ensures homogeneity, preventing side reactions.
   - Physical Integration:
     - Inputs: StartBatch (operator), Temp_PV (sensors), EmergencyStop (safety).
     - Outputs: HeaterOn, AgitatorOn, CoolerOn (relays).
   - Scalability:
     - Add operations (e.g., pressure control) or stages (e.g., separation) as function blocks.
     - Adjust parameters (Temp_SP, Duration) for different batch sizes.
   - Maintenance:
     - Modular function blocks simplify updates and debugging.
     - Monitor Temp_PV, MaintainOp.Timer.ET on HMI for diagnostics.
*)
END_PROGRAM
