(* IEC 61131-3 Structured Text: ISA-88 Cocoa Milk Batch Control *)
(* Purpose: Controls Mixing Process for 100 kg cocoa milk batch *)

(* Function Block: Dose Operation *)
FUNCTION_BLOCK DoseOperation
VAR_INPUT
    Enable : BOOL;                  (* TRUE to run operation *)
    Mass_SP : REAL;                 (* Target mass in kg *)
    Mass_Tol : REAL;                (* Mass tolerance in kg *)
    Mass_PV : REAL;                 (* Measured mass in kg *)
END_VAR
VAR_OUTPUT
    ValveOn : BOOL;                 (* TRUE to open dosing valve *)
    Complete : BOOL;                (* TRUE when dosing complete *)
END_VAR
VAR
    State : INT := 0;               (* 0=Idle, 1=Dosing, 2=Complete *)
END_VAR
IF NOT Enable THEN
    State := 0;
    ValveOn := FALSE;
    Complete := FALSE;
ELSE
    CASE State OF
        0: (* Idle *)
            ValveOn := FALSE;
            Complete := FALSE;
            State := 1;
        1: (* Dosing *)
            ValveOn := TRUE;
            IF ABS(Mass_PV - Mass_SP) <= Mass_Tol THEN
                State := 2;
            END_IF;
        2: (* Complete *)
            ValveOn := FALSE;
            Complete := TRUE;
    END_CASE;
END_IF;
END_FUNCTION_BLOCK

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

(* Function Block: Blend Operation *)
FUNCTION_BLOCK BlendOperation
VAR_INPUT
    Enable : BOOL;                  (* TRUE to run operation *)
    Temp_SP : REAL;                 (* Target temperature in °C *)
    Temp_Tol : REAL;                (* Temperature tolerance in °C *)
    Temp_PV : REAL;                 (* Measured temperature in °C *)
    RPM_SP : REAL;                  (* Target agitator speed in RPM *)
    Duration : TIME;                (* Blending duration *)
END_VAR
VAR_OUTPUT
    HeaterOn : BOOL;                (* TRUE to activate heater *)
    AgitatorOn : BOOL;              (* TRUE to activate agitator *)
    AgitatorSpeed : REAL;           (* Agitator speed in RPM *)
    Complete : BOOL;                (* TRUE when blending complete *)
END_VAR
VAR
    State : INT := 0;               (* 0=Idle, 1=Blending, 2=Complete *)
    BlendTimer : TON;               (* Timer for blending duration *)
END_VAR
IF NOT Enable THEN
    State := 0;
    HeaterOn := FALSE;
    AgitatorOn := FALSE;
    AgitatorSpeed := 0.0;
    Complete := FALSE;
    BlendTimer(IN := FALSE);
ELSE
    CASE State OF
        0: (* Idle *)
            HeaterOn := FALSE;
            AgitatorOn := FALSE;
            AgitatorSpeed := 0.0;
            Complete := FALSE;
            State := 1;
            BlendTimer(IN := TRUE, PT := Duration);
        1: (* Blending *)
            HeaterOn := ABS(Temp_PV - Temp_SP) > Temp_Tol; (* Maintain temp *)
            AgitatorOn := TRUE;
            AgitatorSpeed := RPM_SP;
            IF BlendTimer.Q THEN
                State := 2;
            END_IF;
        2: (* Complete *)
            HeaterOn := FALSE;
            AgitatorOn := FALSE;
            AgitatorSpeed := 0.0;
            Complete := TRUE;
            BlendTimer(IN := FALSE);
    END_CASE;
END_IF;
END_FUNCTION_BLOCK

(* Main Program: Cocoa Milk Batch Control *)
PROGRAM CocoaMilkBatchControl
VAR
    (* Inputs *)
    StartBatch : BOOL;              (* TRUE to start batch *)
    EmergencyStop : BOOL;           (* TRUE to halt operations *)
    MilkMass_PV : REAL;             (* Measured milk mass in kg *)
    WaterMass_PV : REAL;            (* Measured water mass in kg *)
    SugarMass_PV : REAL;            (* Measured liquid sugar mass in kg *)
    CocoaMass_PV : REAL;            (* Measured cocoa mass in kg *)
    TankTemp_PV : REAL;             (* Tank temperature in °C *)
    
    (* Outputs *)
    MilkValveOn : BOOL;             (* TRUE to dose milk *)
    WaterValveOn : BOOL;            (* TRUE to dose water *)
    SugarValveOn : BOOL;            (* TRUE to dose liquid sugar *)
    CocoaValveOn : BOOL;            (* TRUE to dose cocoa *)
    HeaterOn : BOOL;                (* TRUE to activate heater *)
    AgitatorOn : BOOL;              (* TRUE to activate agitator *)
    AgitatorSpeed : REAL;           (* Agitator speed in RPM *)
    BatchComplete : BOOL;           (* TRUE when mixing process complete *)
    
    (* State Machine *)
    State : INT := 0;               (* 0=Idle, 1=DoseMilk, 2=DoseWater, 3=DoseSugar, 4=DoseCocoa, 5=Heat, 6=Blend, 7=Complete *)
    CONSTANT
        STATE_IDLE : INT := 0;      (* Waiting to start *)
        STATE_DOSE_MILK : INT := 1; (* Dosing milk *)
        STATE_DOSE_WATER : INT := 2; (* Dosing water *)
        STATE_DOSE_SUGAR : INT := 3; (* Dosing liquid sugar *)
        STATE_DOSE_COCOA : INT := 4; (* Dosing cocoa *)
        STATE_HEAT : INT := 5;      (* Heating to mixing temp *)
        STATE_BLEND : INT := 6;     (* Blending ingredients *)
        STATE_COMPLETE : INT := 7;  (* Mixing process complete *)
    END_CONSTANT
    
    (* Recipe Parameters *)
    MilkMass_SP : REAL := 60.0;     (* Milk: 60 kg *)
    WaterMass_SP : REAL := 20.0;    (* Water: 20 kg *)
    SugarMass_SP : REAL := 15.0;    (* Liquid sugar: 15 kg *)
    CocoaMass_SP : REAL := 5.0;     (* Cocoa: 5 kg *)
    Mass_Tol : REAL := 0.5;         (* Mass tolerance: ±0.5 kg *)
    MixTemp_SP : REAL := 70.0;      (* Mixing temperature: 70°C *)
    Temp_Tol : REAL := 2.0;         (* Temperature tolerance: ±2°C *)
    MixRPM_SP : REAL := 200.0;      (* Agitator speed: 200 RPM *)
    MixDuration : TIME := T#10m;    (* Blending duration: 10 minutes *)
    
    (* Operation Instances *)
    DoseMilkOp : DoseOperation;     (* Dosing milk *)
    DoseWaterOp : DoseOperation;    (* Dosing water *)
    DoseSugarOp : DoseOperation;    (* Dosing liquid sugar *)
    DoseCocoaOp : DoseOperation;    (* Dosing cocoa *)
    HeatOp : HeatOperation;         (* Heating operation *)
    BlendOp : BlendOperation;       (* Blending operation *)
END_VAR

(* Main Logic *)
IF EmergencyStop THEN
    State := STATE_IDLE;
    MilkValveOn := FALSE;
    WaterValveOn := FALSE;
    SugarValveOn := FALSE;
    CocoaValveOn := FALSE;
    HeaterOn := FALSE;
    AgitatorOn := FALSE;
    AgitatorSpeed := 0.0;
    BatchComplete := FALSE;
    DoseMilkOp.Enable := FALSE;
    DoseWaterOp.Enable := FALSE;
    DoseSugarOp.Enable := FALSE;
    DoseCocoaOp.Enable := FALSE;
    HeatOp.Enable := FALSE;
    BlendOp.Enable := FALSE;
ELSIF StartBatch AND State = STATE_IDLE THEN
    State := STATE_DOSE_MILK;
    BatchComplete := FALSE;
END_IF;

(* State Machine *)
CASE State OF
    STATE_IDLE:
        MilkValveOn := FALSE;
        WaterValveOn := FALSE;
        SugarValveOn := FALSE;
        CocoaValveOn := FALSE;
        HeaterOn := FALSE;
        AgitatorOn := FALSE;
        AgitatorSpeed := 0.0;
        BatchComplete := FALSE;
        DoseMilkOp.Enable := FALSE;
        DoseWaterOp.Enable := FALSE;
        DoseSugarOp.Enable := FALSE;
        DoseCocoaOp.Enable := FALSE;
        HeatOp.Enable := FALSE;
        BlendOp.Enable := FALSE;

    STATE_DOSE_MILK:
        DoseMilkOp(Enable := TRUE, Mass_SP := MilkMass_SP, Mass_Tol := Mass_Tol, Mass_PV := MilkMass_PV);
        MilkValveOn := DoseMilkOp.ValveOn;
        IF DoseMilkOp.Complete THEN
            State := STATE_DOSE_WATER;
            DoseMilkOp.Enable := FALSE;
        END_IF;

    STATE_DOSE_WATER:
        DoseWaterOp(Enable := TRUE, Mass_SP := WaterMass_SP, Mass_Tol := Mass_Tol, Mass_PV := WaterMass_PV);
        WaterValveOn := DoseWaterOp.ValveOn;
        IF DoseWaterOp.Complete THEN
            State := STATE_DOSE_SUGAR;
            DoseWaterOp.Enable := FALSE;
        END_IF;

    STATE_DOSE_SUGAR:
        DoseSugarOp(Enable := TRUE, Mass_SP := SugarMass_SP, Mass_Tol := Mass_Tol, Mass_PV := SugarMass_PV);
        SugarValveOn := DoseSugarOp.ValveOn;
        IF DoseSugarOp.Complete THEN
            State := STATE_DOSE_COCOA;
            DoseSugarOp.Enable := FALSE;
        END_IF;

    STATE_DOSE_COCOA:
        DoseCocoaOp(Enable := TRUE, Mass_SP := CocoaMass_SP, Mass_Tol := Mass_Tol, Mass_PV := CocoaMass_PV);
        CocoaValveOn := DoseCocoaOp.ValveOn;
        IF DoseCocoaOp.Complete THEN
            State := STATE_HEAT;
            DoseCocoaOp.Enable := FALSE;
        END_IF;

    STATE_HEAT:
        HeatOp(Enable := TRUE, Temp_SP := MixTemp_SP, Temp_Tol := Temp_Tol, Temp_PV := TankTemp_PV);
        HeaterOn := HeatOp.HeaterOn;
        IF HeatOp.Complete THEN
            State := STATE_BLEND;
            HeatOp.Enable := FALSE;
        END_IF;

    STATE_BLEND:
        BlendOp(Enable := TRUE, Temp_SP := MixTemp_SP, Temp_Tol := Temp_Tol, Temp_PV := TankTemp_PV, RPM_SP := MixRPM_SP, Duration := MixDuration);
        HeaterOn := BlendOp.HeaterOn;
        AgitatorOn := BlendOp.AgitatorOn;
        AgitatorSpeed := BlendOp.AgitatorSpeed;
        IF BlendOp.Complete THEN
            State := STATE_COMPLETE;
            BlendOp.Enable := FALSE;
        END_IF;

    STATE_COMPLETE:
        MilkValveOn := FALSE;
        WaterValveOn := FALSE;
        SugarValveOn := FALSE;
        CocoaValveOn := FALSE;
        HeaterOn := FALSE;
        AgitatorOn := FALSE;
        AgitatorSpeed := 0.0;
        BatchComplete := TRUE;
        IF NOT StartBatch THEN
            State := STATE_IDLE;
        END_IF;

ELSE
    State := STATE_IDLE; (* Fallback to safe state *)
END_CASE;

(* Notes:
   - ISA-88 Compliance:
     - Unit Procedure: Mixing Process (B), with operations for dosing, heating, blending.
     - Operations: Dose, Heat, Blend implemented as reusable function blocks.
     - Recipe Parameters: Mass_SP, Temp_SP, RPM_SP, Duration for flexibility.
     - State Model: Idle, Running (Dose/Heat/Blend), Complete per ISA-88.
   - Batch Formula (100 kg):
     - Milk: 60 kg, Water: 20 kg, Liquid Sugar: 15 kg, Cocoa: 5 kg (±0.5 kg).
   - Mixing Process:
     1. Dose: Sequentially add ingredients to ±0.5 kg accuracy.
     2. Heat: Reach 70°C ± 2°C for dissolution and pasteurization.
     3. Blend: Mix at 200 RPM for 10 min to ensure homogeneity.
   - Safety:
     - EmergencyStop halts all operations, resets state.
     - Tolerances (0.5 kg, 2°C) ensure quality and consistency.
   - Quality:
     - Accurate dosing prevents flavor imbalance.
     - Stable 70°C and 200 RPM ensure uniform texture and safety.
   - Physical Integration:
     - Inputs: StartBatch (operator), Mass_PV, Temp_PV (sensors), EmergencyStop.
     - Outputs: ValveOn, HeaterOn, AgitatorOn (relays), AgitatorSpeed (analog).
   - Scalability:
     - Adjust Mass_SP, Duration for larger batches.
     - Add operations (e.g., cooling) as new function blocks.
   - Maintenance:
     - Modular function blocks simplify debugging.
     - Monitor Mass_PV, Temp_PV, BlendOp.BlendTimer.ET on HMI.
*)
END_PROGRAM
