(* IEC 61131-3 Structured Text: ISA-88 PVC Batch Control *)
(* Purpose: Controls Polymerization, Decovar, and Dry stages for PVC production *)

(* Function Block: Evacuate Reactor *)
FUNCTION_BLOCK EvacuateReactor
VAR_INPUT
    Enable : BOOL;                  (* TRUE to run operation *)
    Pressure_SP : REAL;             (* Target pressure in bar *)
    Pressure_Tol : REAL;            (* Pressure tolerance in bar *)
    Pressure_PV : REAL;             (* Measured pressure in bar *)
END_VAR
VAR_OUTPUT
    VacuumPumpOn : BOOL;            (* TRUE to activate vacuum pump *)
    Complete : BOOL;                (* TRUE when pressure reached *)
END_VAR
VAR
    State : INT := 0;               (* 0=Idle, 1=Evacuating, 2=Complete *)
END_VAR
IF NOT Enable THEN
    State := 0;
    VacuumPumpOn := FALSE;
    Complete := FALSE;
ELSE
    CASE State OF
        0: (* Idle *)
            VacuumPumpOn := FALSE;
            Complete := FALSE;
            State := 1;
        1: (* Evacuating *)
            VacuumPumpOn := TRUE;
            IF ABS(Pressure_PV - Pressure_SP) <= Pressure_Tol THEN
                State := 2;
            END_IF;
        2: (* Complete *)
            VacuumPumpOn := FALSE;
            Complete := TRUE;
    END_CASE;
END_IF;
END_FUNCTION_BLOCK

(* Function Block: Dose Material *)
FUNCTION_BLOCK DoseMaterial
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

(* Function Block: Dose Water *)
FUNCTION_BLOCK DoseWater
VAR_INPUT
    Enable : BOOL;                  (* TRUE to run operation *)
    Volume_SP : REAL;               (* Target volume in liters *)
    Volume_Tol : REAL;              (* Volume tolerance in liters *)
    Volume_PV : REAL;               (* Measured volume in liters *)
END_VAR
VAR_OUTPUT
    ValveOn : BOOL;                 (* TRUE to open water valve *)
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
            IF ABS(Volume_PV - Volume_SP) <= Volume_Tol THEN
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
    Temp_Min : REAL;                (* Minimum temperature in °C *)
    Temp_Max : REAL;                (* Maximum temperature in °C *)
    Temp_PV : REAL;                 (* Measured temperature in °C *)
END_VAR
VAR_OUTPUT
    HeaterOn : BOOL;                (* TRUE to activate heater *)
    Complete : BOOL;                (* TRUE when temperature in range *)
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
            HeaterOn := Temp_PV < Temp_Max;
            IF Temp_PV >= Temp_Min AND Temp_PV <= Temp_Max THEN
                State := 2;
            END_IF;
        2: (* Complete *)
            HeaterOn := FALSE;
            Complete := TRUE;
    END_CASE;
END_IF;
END_FUNCTION_BLOCK

(* Function Block: Polymerization *)
FUNCTION_BLOCK PolymerizeOperation
VAR_INPUT
    Enable : BOOL;                  (* TRUE to run operation *)
    Temp_Min : REAL;                (* Minimum temperature in °C *)
    Temp_Max : REAL;                (* Maximum temperature in °C *)
    Temp_PV : REAL;                 (* Measured temperature in °C *)
    Pressure_SP : REAL;             (* Target pressure for completion in bar *)
    Pressure_Tol : REAL;            (* Pressure tolerance in bar *)
    Pressure_PV : REAL;             (* Measured pressure in bar *)
END_VAR
VAR_OUTPUT
    HeaterOn : BOOL;                (* TRUE to activate heater *)
    AgitatorOn : BOOL;              (* TRUE to activate agitator *)
    Complete : BOOL;                (* TRUE when pressure drops *)
END_VAR
VAR
    State : INT := 0;               (* 0=Idle, 1=Reacting, 2=Complete *)
END_VAR
IF NOT Enable THEN
    State := 0;
    HeaterOn := FALSE;
    AgitatorOn := FALSE;
    Complete := FALSE;
ELSE
    CASE State OF
        0: (* Idle *)
            HeaterOn := FALSE;
            AgitatorOn := FALSE;
            Complete := FALSE;
            State := 1;
        1: (* Reacting *)
            HeaterOn := Temp_PV < Temp_Max;
            AgitatorOn := TRUE;
            IF ABS(Pressure_PV - Pressure_SP) <= Pressure_Tol THEN
                State := 2;
            END_IF;
        2: (* Complete *)
            HeaterOn := FALSE;
            AgitatorOn := FALSE;
            Complete := TRUE;
    END_CASE;
END_IF;
END_FUNCTION_BLOCK

(* Function Block: Vent VCM *)
FUNCTION_BLOCK VentVCM
VAR_INPUT
    Enable : BOOL;                  (* TRUE to run operation *)
    Duration : TIME;                (* Venting duration *)
END_VAR
VAR_OUTPUT
    VentValveOn : BOOL;             (* TRUE to open vent valve *)
    Complete : BOOL;                (* TRUE when venting complete *)
END_VAR
VAR
    State : INT := 0;               (* 0=Idle, 1=Venting, 2=Complete *)
    VentTimer : TON;                (* Timer for venting *)
END_VAR
IF NOT Enable THEN
    State := 0;
    VentValveOn := FALSE;
    Complete := FALSE;
    VentTimer(IN := FALSE);
ELSE
    CASE State OF
        0: (* Idle *)
            VentValveOn := FALSE;
            Complete := FALSE;
            State := 1;
            VentTimer(IN := TRUE, PT := Duration);
        1: (* Venting *)
            VentValveOn := TRUE;
            IF VentTimer.Q THEN
                State := 2;
            END_IF;
        2: (* Complete *)
            VentValveOn := FALSE;
            Complete := TRUE;
            VentTimer(IN := FALSE);
    END_CASE;
END_IF;
END_FUNCTION_BLOCK

(* Function Block: Dry Operation *)
FUNCTION_BLOCK DryOperation
VAR_INPUT
    Enable : BOOL;                  (* TRUE to run operation *)
    Temp_SP : REAL;                 (* Target temperature in °C *)
    Temp_Tol : REAL;                (* Temperature tolerance in °C *)
    Temp_PV : REAL;                 (* Measured temperature in °C *)
    Duration : TIME;                (* Drying duration *)
END_VAR
VAR_OUTPUT
    HeaterOn : BOOL;                (* TRUE to activate heater *)
    Complete : BOOL;                (* TRUE when drying complete *)
END_VAR
VAR
    State : INT := 0;               (* 0=Idle, 1=Heating, 2=Drying, 3=Complete *)
    DryTimer : TON;                 (* Timer for drying *)
END_VAR
IF NOT Enable THEN
    State := 0;
    HeaterOn := FALSE;
    Complete := FALSE;
    DryTimer(IN := FALSE);
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
                DryTimer(IN := TRUE, PT := Duration);
            END_IF;
        2: (* Drying *)
            HeaterOn := ABS(Temp_PV - Temp_SP) > Temp_Tol;
            IF DryTimer.Q THEN
                State := 3;
            END_IF;
        3: (* Complete *)
            HeaterOn := FALSE;
            Complete := TRUE;
            DryTimer(IN := FALSE);
    END_CASE;
END_IF;
END_FUNCTION_BLOCK

(* Main Program: PVC Batch Control *)
PROGRAM PVCBatchControl
VAR
    (* Inputs *)
    StartBatch : BOOL;              (* TRUE to start batch *)
    EmergencyStop : BOOL;           (* TRUE to halt operations *)
    ReactorPressure_PV : REAL;      (* Reactor pressure in bar *)
    ReactorTemp_PV : REAL;          (* Reactor temperature in °C *)
    WaterVolume_PV : REAL;          (* Measured water volume in liters *)
    SurfactantMass_PV : REAL;       (* Measured surfactant mass in kg *)
    VCMMass_PV : REAL;              (* Measured VCM mass in kg *)
    CatalystMass_PV : REAL;         (* Measured catalyst mass in kg *)
    DryerTemp_PV : REAL;            (* Dryer temperature in °C *)
    
    (* Outputs *)
    VacuumPumpOn : BOOL;            (* TRUE to activate vacuum pump *)
    WaterValveOn : BOOL;            (* TRUE to dose water *)
    SurfactantValveOn : BOOL;       (* TRUE to dose surfactant *)
    VCMValveOn : BOOL;              (* TRUE to dose VCM *)
    CatalystValveOn : BOOL;         (* TRUE to dose catalyst *)
    ReactorHeaterOn : BOOL;         (* TRUE to activate reactor heater *)
    ReactorAgitatorOn : BOOL;       (* TRUE to activate agitator *)
    VentValveOn : BOOL;             (* TRUE to open vent valve *)
    TransferPumpOn : BOOL;          (* TRUE to transfer slurry *)
    DryerHeaterOn : BOOL;           (* TRUE to activate dryer heater *)
    BatchComplete : BOOL;           (* TRUE when batch complete *)
    
    (* State Machine *)
    State : INT := 0;               (* 0=Idle, 1=Polymerize, 2=Decovar, 3=Dry, 4=Complete *)
    CONSTANT
        STATE_IDLE : INT := 0;      (* Waiting to start *)
        STATE_POLYMERIZE : INT := 1; (* Polymerization stage *)
        STATE_DECOVAR : INT := 2;   (* Decovar stage *)
        STATE_DRY : INT := 3;       (* Drying stage *)
        STATE_COMPLETE : INT := 4;  (* Batch complete *)
    END_CONSTANT
    
    (* Sub-State for Polymerization *)
    PolySubState : INT := 0;        (* 0=Idle, 1=Evacuate, ..., 6=Polymerize *)
    CONSTANT
        POLY_IDLE : INT := 0;
        POLY_EVACUATE : INT := 1;
        POLY_DOSE_WATER : INT := 2;
        POLY_DOSE_SURFACTANT : INT := 3;
        POLY_DOSE_VCM : INT := 4;
        POLY_DOSE_CATALYST : INT := 5;
        POLY_HEAT : INT := 6;
        POLY_REACT : INT := 7;
        POLY_COMPLETE : INT := 8;
    END_CONSTANT
    
    (* Sub-State for Decovar *)
    DecovarSubState : INT := 0;     (* 0=Idle, 1=Vent, 2=Transfer, 3=Complete *)
    CONSTANT
        DECOVAR_IDLE : INT := 0;
        DECOVAR_VENT : INT := 1;
        DECOVAR_TRANSFER : INT := 2;
        DECOVAR_COMPLETE : INT := 3;
    END_CONSTANT
    
    (* Recipe Parameters *)
    EvacPressure_SP : REAL := 0.1;  (* Evacuation pressure: 0.1 bar *)
    Pressure_Tol : REAL := 0.05;    (* Pressure tolerance: ±0.05 bar *)
    WaterVolume_SP : REAL := 1000.0; (* Water: 1000 L *)
    Volume_Tol : REAL := 10.0;      (* Volume tolerance: ±10 L *)
    SurfactantMass_SP : REAL := 5.0; (* Surfactant: 5 kg *)
    VCMMass_SP : REAL := 500.0;     (* VCM: 500 kg *)
    CatalystMass_SP : REAL := 0.5;  (* Catalyst: 0.5 kg *)
    Mass_Tol : REAL := 0.5;         (* Mass tolerance: ±0.5 kg *)
    ReactionTemp_Min : REAL := 55.0; (* Reaction temp: 55–60°C *)
    ReactionTemp_Max : REAL := 60.0;
    ReactionPressure_SP : REAL := 2.0; (* Reaction completion: 2 bar *)
    VentDuration : TIME := T#5m;    (* Venting duration: 5 min *)
    DryerTemp_SP : REAL := 70.0;    (* Drying temp: 70°C *)
    DryerTemp_Tol : REAL := 3.0;    (* Drying tolerance: ±3°C *)
    DryDuration : TIME := T#3h;     (* Drying duration: 3 hours *)
    
    (* Operation Instances *)
    EvacOp : EvacuateReactor;       (* Evacuate reactor *)
    DoseWaterOp : DoseWater;        (* Dose water *)
    DoseSurfactantOp : DoseMaterial; (* Dose surfactant *)
    DoseVCMOp : DoseMaterial;       (* Dose VCM *)
    DoseCatalystOp : DoseMaterial;  (* Dose catalyst *)
    HeatOp : HeatOperation;         (* Heat operation *)
    PolyOp : PolymerizeOperation;   (* Polymerization *)
    VentOp : VentVCM;               (* Vent VCM *)
    DryOp : DryOperation;           (* Dry operation *)
END_VAR

(* Main Logic *)
IF EmergencyStop THEN
    State := STATE_IDLE;
    PolySubState := POLY_IDLE;
    DecovarSubState := DECOVAR_IDLE;
    VacuumPumpOn := FALSE;
    WaterValveOn := FALSE;
    SurfactantValveOn := FALSE;
    VCMValveOn := FALSE;
    CatalystValveOn := FALSE;
    ReactorHeaterOn := FALSE;
    ReactorAgitatorOn := FALSE;
    VentValveOn := FALSE;
    TransferPumpOn := FALSE;
    DryerHeaterOn := FALSE;
    BatchComplete := FALSE;
    EvacOp.Enable := FALSE;
    DoseWaterOp.Enable := FALSE;
    DoseSurfactantOp.Enable := FALSE;
    DoseVCMOp.Enable := FALSE;
    DoseCatalystOp.Enable := FALSE;
    HeatOp.Enable := FALSE;
    PolyOp.Enable := FALSE;
    VentOp.Enable := FALSE;
    DryOp.Enable := FALSE;
ELSIF StartBatch AND State = STATE_IDLE THEN
    State := STATE_POLYMERIZE;
    PolySubState := POLY_EVACUATE;
    BatchComplete := FALSE;
END_IF;

(* State Machine *)
CASE State OF
    STATE_IDLE:
        VacuumPumpOn := FALSE;
        WaterValveOn := FALSE;
        SurfactantValveOn := FALSE;
        VCMValveOn := FALSE;
        CatalystValveOn := FALSE;
        ReactorHeaterOn := FALSE;
        ReactorAgitatorOn := FALSE;
        VentValveOn := FALSE;
        TransferPumpOn := FALSE;
        DryerHeaterOn := FALSE;
        BatchComplete := FALSE;
        PolySubState := POLY_IDLE;
        DecovarSubState := DECOVAR_IDLE;
        EvacOp.Enable := FALSE;
        DoseWaterOp.Enable := FALSE;
        DoseSurfactantOp.Enable := FALSE;
        DoseVCMOp.Enable := FALSE;
        DoseCatalystOp.Enable := FALSE;
        HeatOp.Enable := FALSE;
        PolyOp.Enable := FALSE;
        VentOp.Enable := FALSE;
        DryOp.Enable := FALSE;

    STATE_POLYMERIZE:
        CASE PolySubState OF
            POLY_IDLE:
                PolySubState := POLY_EVACUATE;
            POLY_EVACUATE:
                EvacOp(Enable := TRUE, Pressure_SP := EvacPressure_SP, Pressure_Tol := Pressure_Tol, Pressure_PV := ReactorPressure_PV);
                VacuumPumpOn := EvacOp.VacuumPumpOn;
                IF EvacOp.Complete THEN
                    PolySubState := POLY_DOSE_WATER;
                    EvacOp.Enable := FALSE;
                END_IF;
            POLY_DOSE_WATER:
                DoseWaterOp(Enable := TRUE, Volume_SP := WaterVolume_SP, Volume_Tol := Volume_Tol, Volume_PV := WaterVolume_PV);
                WaterValveOn := DoseWaterOp.ValveOn;
                IF DoseWaterOp.Complete THEN
                    PolySubState := POLY_DOSE_SURFACTANT;
                    DoseWaterOp.Enable := FALSE;
                END_IF;
            POLY_DOSE_SURFACTANT:
                DoseSurfactantOp(Enable := TRUE, Mass_SP := SurfactantMass_SP, Mass_Tol := Mass_Tol, Mass_PV := SurfactantMass_PV);
                SurfactantValveOn := DoseSurfactantOp.ValveOn;
                IF DoseSurfactantOp.Complete THEN
                    PolySubState := POLY_DOSE_VCM;
                    DoseSurfactantOp.Enable := FALSE;
                END_IF;
            POLY_DOSE_VCM:
                DoseVCMOp(Enable := TRUE, Mass_SP := VCMMass_SP, Mass_Tol := Mass_Tol, Mass_PV := VCMMass_PV);
                VCMValveOn := DoseVCMOp.ValveOn;
                IF DoseVCMOp.Complete THEN
                    PolySubState := POLY_DOSE_CATALYST;
                    DoseVCMOp.Enable := FALSE;
                END_IF;
            POLY_DOSE_CATALYST:
                DoseCatalystOp(Enable := TRUE, Mass_SP := CatalystMass_SP, Mass_Tol := Mass_Tol, Mass_PV := CatalystMass_PV);
                CatalystValveOn := DoseCatalystOp.ValveOn;
                IF DoseCatalystOp.Complete THEN
                    PolySubState := POLY_HEAT;
                    DoseCatalystOp.Enable := FALSE;
                END_IF;
            POLY_HEAT:
                HeatOp(Enable := TRUE, Temp_Min := ReactionTemp_Min, Temp_Max := ReactionTemp_Max, Temp_PV := ReactorTemp_PV);
                ReactorHeaterOn := HeatOp.HeaterOn;
                IF HeatOp.Complete THEN
                    PolySubState := POLY_REACT;
                    HeatOp.Enable := FALSE;
                END_IF;
            POLY_REACT:
                PolyOp(Enable := TRUE, Temp_Min := ReactionTemp_Min, Temp_Max := ReactionTemp_Max, Temp_PV := ReactorTemp_PV, Pressure_SP := ReactionPressure_SP, Pressure_Tol := Pressure_Tol, Pressure_PV := ReactorPressure_PV);
                ReactorHeaterOn := PolyOp.HeaterOn;
                ReactorAgitatorOn := PolyOp.AgitatorOn;
                IF PolyOp.Complete THEN
                    PolySubState := POLY_COMPLETE;
                    PolyOp.Enable := FALSE;
                END_IF;
            POLY_COMPLETE:
                ReactorHeaterOn := FALSE;
                ReactorAgitatorOn := FALSE;
                State := STATE_DECOVAR;
                DecovarSubState := DECOVAR_VENT;
        END_CASE;

    STATE_DECOVAR:
        CASE DecovarSubState OF
            DECOVAR_IDLE:
                DecovarSubState := DECOVAR_VENT;
            DECOVAR_VENT:
                VentOp(Enable := TRUE, Duration := VentDuration);
                VentValveOn := VentOp.VentValveOn;
                IF VentOp.Complete THEN
                    DecovarSubState := DECOVAR_TRANSFER;
                    VentOp.Enable := FALSE;
                END_IF;
            DECOVAR_TRANSFER:
                TransferPumpOn := TRUE;
                IF ReactorPressure_PV < 0.5 THEN (* Assume transfer complete *)
                    DecovarSubState := DECOVAR_COMPLETE;
                    TransferPumpOn := FALSE;
                END_IF;
            DECOVAR_COMPLETE:
                TransferPumpOn := FALSE;
                State := STATE_DRY;
        END_CASE;

    STATE_DRY:
        DryOp(Enable := TRUE, Temp_SP := DryerTemp_SP, Temp_Tol := DryerTemp_Tol, Temp_PV := DryerTemp_PV, Duration := DryDuration);
        DryerHeaterOn := DryOp.HeaterOn;
        IF DryOp.Complete THEN
            State := STATE_COMPLETE;
            DryOp.Enable := FALSE;
        END_IF;

    STATE_COMPLETE:
        VacuumPumpOn := FALSE;
        WaterValveOn := FALSE;
        SurfactantValveOn := FALSE;
        VCMValveOn := FALSE;
        CatalystValveOn := FALSE;
        ReactorHeaterOn := FALSE;
        ReactorAgitatorOn := FALSE;
        VentValveOn := FALSE;
        TransferPumpOn := FALSE;
        DryerHeaterOn := FALSE;
        BatchComplete := TRUE;
        IF NOT StartBatch THEN
            State := STATE_IDLE;
        END_IF;

ELSE
    State := STATE_IDLE; (* Fallback to safe state *)
END_CASE;

(* Notes:
   - ISA-88 Compliance:
     - Unit Procedures: Polymerize, Decovar, Dry, each with modular operations.
     - Operations: Evacuate, Dose, Heat, Polymerize, Vent, Dry as function blocks.
     - Recipe Parameters: Temp_SP, Pressure_SP, Volume_SP, etc., for flexibility.
     - State Model: Idle, Running, Complete per ISA-88.
   - Process Stages:
     - Polymerization: Evacuate (<0.1 bar), dose water (1000 L), surfactant (5 kg), VCM (500 kg), catalyst (0.5 kg), heat to 55–60°C, react until 2 bar.
     - Decovar: Vent VCM (5 min), transfer slurry.
     - Dry: Dry at 70°C ± 3°C for 3 hours.
   - Safety:
     - EmergencyStop halts all operations, resets state.
     - Tolerances (0.05 bar, 10 L, 0.5 kg, 3°C) ensure quality.
   - Quality:
     - Precise dosing and temperature (55–60°C) ensure consistent polymerization.
     - Pressure-based completion (2 bar) confirms reaction endpoint.
   - Physical Integration:
     - Inputs: StartBatch, Pressure_PV, Temp_PV, Mass_PV, Volume_PV (sensors).
     - Outputs: ValveOn, PumpOn, HeaterOn, AgitatorOn (relays).
   - Scalability:
     - Adjust parameters (Volume_SP, Mass_SP) for larger reactors.
     - Add operations (e.g., cooling) as function blocks.
   - Maintenance:
     - Modular function blocks simplify updates.
     - Monitor Temp_PV, Pressure_PV, DryOp.DryTimer.ET on HMI.
*)
END_PROGRAM
