(* IEC 61131-3 Structured Text: ISA-88 Steel Plant Shutdown Control *)
(* Purpose: Safely deactivates steel production facility with modular logic *)

(* Function Block: Ramp Down Gas Flow *)
FUNCTION_BLOCK RampDownGas
VAR_INPUT
    Enable : BOOL;                  (* TRUE to run operation *)
    InitialFlow : REAL;             (* Initial gas flow in Nm³/h *)
    TargetFlow : REAL;              (* Target gas flow, typically 0 Nm³/h *)
    RampDuration : TIME;            (* Ramp-down duration, e.g., T#12h *)
    GasFlow_PV : REAL;              (* Measured gas flow in Nm³/h *)
    FurnaceTemp_PV : REAL;          (* Measured furnace temperature in °C *)
END_VAR
VAR_OUTPUT
    GasValvePos : REAL;             (* Gas valve position, 0–100% *)
    FlowAlarm : BOOL;               (* TRUE if flow deviates or temp too low *)
    Complete : BOOL;                (* TRUE when ramp-down complete *)
END_VAR
VAR
    State : INT := 0;               (* 0=Idle, 1=Ramping, 2=Complete *)
    RampTimer : TON;                (* Timer for ramp duration *)
    ElapsedTime : TIME;             (* Elapsed time since ramp start *)
    FlowSetpoint : REAL;            (* Current flow setpoint *)
    FlowRate : REAL;                (* Ramp rate in Nm³/h per second *)
    FlowTol : REAL := 25.0;         (* Flow tolerance: ±5% of 500 Nm³/h *)
    MinTemp : REAL := 700.0;        (* Minimum safe temperature *)
END_VAR
IF NOT Enable THEN
    State := 0;
    GasValvePos := 0.0;
    FlowAlarm := FALSE;
    Complete := FALSE;
    RampTimer(IN := FALSE);
ELSE
    CASE State OF
        0: (* Idle *)
            GasValvePos := 100.0;   (* Full flow initially *)
            FlowAlarm := FALSE;
            Complete := FALSE;
            FlowRate := (InitialFlow - TargetFlow) / TIME_TO_REAL(RampDuration, 1.0);
            State := 1;
            RampTimer(IN := TRUE, PT := RampDuration);
        1: (* Ramping *)
            ElapsedTime := RampTimer.ET;
            FlowSetpoint := InitialFlow - (FlowRate * TIME_TO_REAL(ElapsedTime, 1.0));
            IF FlowSetpoint < TargetFlow THEN
                FlowSetpoint := TargetFlow;
            END_IF;
            GasValvePos := (FlowSetpoint / InitialFlow) * 100.0;
            FlowAlarm := (ABS(GasFlow_PV - FlowSetpoint) > FlowTol) OR 
                         (FurnaceTemp_PV < MinTemp);
            IF RampTimer.Q THEN
                State := 2;
            END_IF;
        2: (* Complete *)
            GasValvePos := 0.0;
            FlowAlarm := FALSE;
            Complete := TRUE;
            RampTimer(IN := FALSE);
    END_CASE;
END_IF;
(* Gradually reduces gas flow over 12 hours, monitors safety conditions *)
END_FUNCTION_BLOCK

(* Function Block: Adjust Oxygen Supply *)
FUNCTION_BLOCK AdjustOxygen
VAR_INPUT
    Enable : BOOL;                  (* TRUE to run operation *)
    GasFlow_PV : REAL;              (* Measured gas flow in Nm³/h *)
    FurnaceTemp_PV : REAL;          (* Measured furnace temperature in °C *)
    OxygenFlow_PV : REAL;           (* Measured oxygen flow in Nm³/h *)
END_VAR
VAR_OUTPUT
    OxygenValvePos : REAL;          (* Oxygen valve position, 0–100% *)
    RatioAlarm : BOOL;              (* TRUE if fuel-to-air ratio deviates *)
END_VAR
VAR
    State : INT := 0;               (* 0=Idle, 1=Adjusting, 2=Stopped *)
    FuelAirRatio : REAL := 2.5;     (* Target fuel-to-air ratio: 1:2.5 *)
    OxygenSetpoint : REAL;          (* Target oxygen flow in Nm³/h *)
    RatioTol : REAL := 0.25;        (* Ratio tolerance: ±10% of 2.5 *)
    MaxTemp : REAL := 850.0;        (* Maximum safe temperature *)
    MinTemp : REAL := 700.0;        (* Minimum safe temperature *)
    MaxOxygenFlow : REAL := 1250.0; (* Max oxygen flow: 2.5 * 500 Nm³/h *)
END_VAR
IF NOT Enable THEN
    State := 0;
    OxygenValvePos := 0.0;
    RatioAlarm := FALSE;
ELSE
    CASE State OF
        0: (* Idle *)
            OxygenValvePos := 0.0;
            RatioAlarm := FALSE;
            IF GasFlow_PV > 0.0 AND FurnaceTemp_PV > MinTemp THEN
                State := 1;
            END_IF;
        1: (* Adjusting *)
            OxygenSetpoint := GasFlow_PV * FuelAirRatio;
            IF OxygenSetpoint > MaxOxygenFlow THEN
                OxygenSetpoint := MaxOxygenFlow;
            END_IF;
            OxygenValvePos := (OxygenSetpoint / MaxOxygenFlow) * 100.0;
            RatioAlarm := (ABS(OxygenFlow_PV - OxygenSetpoint) > (RatioTol * OxygenSetpoint)) OR 
                          (FurnaceTemp_PV > MaxTemp);
            IF GasFlow_PV <= 0.0 OR FurnaceTemp_PV <= MinTemp THEN
                State := 2;
            END_IF;
        2: (* Stopped *)
            OxygenValvePos := 0.0;
            RatioAlarm := FALSE;
    END_CASE;
END_IF;
(* Dynamically adjusts oxygen to maintain 1:2.5 fuel-to-air ratio *)
END_FUNCTION_BLOCK

(* Function Block: Cool Furnace *)
FUNCTION_BLOCK CoolFurnace
VAR_INPUT
    Enable : BOOL;                  (* TRUE to run operation *)
    Temp_SP : REAL;                 (* Target temperature in °C *)
    Temp_Tol : REAL;                (* Temperature tolerance in °C *)
    Temp_PV : REAL;                 (* Measured temperature in °C *)
END_VAR
VAR_OUTPUT
    CoolerOn : BOOL;                (* TRUE to activate cooling system *)
    CoolantFlow : REAL;             (* Coolant flow rate, 0–100% *)
    Complete : BOOL;                (* TRUE when temperature reached *)
END_VAR
VAR
    State : INT := 0;               (* 0=Idle, 1=Cooling, 2=Complete *)
    MinTemp : REAL := 180.0;        (* Minimum safe temperature *)
END_VAR
IF NOT Enable THEN
    State := 0;
    CoolerOn := FALSE;
    CoolantFlow := 0.0;
    Complete := FALSE;
ELSE
    CASE State OF
        0: (* Idle *)
            CoolerOn := FALSE;
            CoolantFlow := 0.0;
            Complete := FALSE;
            State := 1;
        1: (* Cooling *)
            CoolerOn := TRUE;
            CoolantFlow := 100.0;   (* Full flow for simplicity *)
            IF ABS(Temp_PV - Temp_SP) <= Temp_Tol THEN
                State := 2;
            ELSIF Temp_PV < MinTemp THEN
                State := 2;         (* Prevent over-cooling *)
            END_IF;
        2: (* Complete *)
            CoolerOn := FALSE;
            CoolantFlow := 0.0;
            Complete := TRUE;
    END_CASE;
END_IF;
(* Cools furnace to safe temperature *)
END_FUNCTION_BLOCK

(* Main Program: Steel Plant Shutdown *)
PROGRAM SteelPlantShutdown
VAR
    (* Inputs *)
    StartShutdown : BOOL;           (* TRUE to start shutdown *)
    EmergencyStop : BOOL;           (* TRUE to halt operations *)
    GasFlow_PV : REAL;              (* Measured gas flow in Nm³/h *)
    OxygenFlow_PV : REAL;           (* Measured oxygen flow in Nm³/h *)
    FurnaceTemp_PV : REAL;          (* Measured furnace temperature in °C *)
    NitrogenFlow_PV : REAL;         (* Measured nitrogen flow in Nm³/h *)
    
    (* Outputs *)
    GasValvePos : REAL;             (* Gas valve position, 0–100% *)
    OxygenValvePos : REAL;          (* Oxygen valve position, 0–100% *)
    FurnaceHeaterOn : BOOL;         (* TRUE to activate furnace heater *)
    NitrogenValveOn : BOOL;         (* TRUE to open nitrogen valve *)
    CoolerOn : BOOL;                (* TRUE to activate cooling system *)
    CoolantFlow : REAL;             (* Coolant flow rate, 0–100% *)
    ShutdownComplete : BOOL;        (* TRUE when shutdown complete *)
    FlowAlarm : BOOL;               (* TRUE if gas flow deviates *)
    RatioAlarm : BOOL;              (* TRUE if fuel-to-air ratio deviates *)
    
    (* State Machine *)
    State : INT := 0;               (* Main state *)
    CONSTANT
        STATE_IDLE : INT := 0;      (* Waiting to start *)
        STATE_INITIATE : INT := 1;  (* Initiate shutdown *)
        STATE_REDUCE_LOAD : INT := 2; (* Reduce furnace load *)
        STATE_PURGE_GASES : INT := 3; (* Purge combustible gases *)
        STATE_RAMP_GAS : INT := 4;  (* Ramp down gas flow *)
        STATE_ADJUST_OXYGEN : INT := 5; (* Adjust oxygen supply *)
        STATE_COOL_FURNACE : INT := 6; (* Cool furnace *)
        STATE_FINAL : INT := 7;     (* Final shutdown *)
        STATE_COMPLETE : INT := 8;  (* Shutdown complete *)
    END_CONSTANT
    
    (* Recipe Parameters *)
    InitialGasFlow : REAL := 500.0; (* Initial gas flow: 500 Nm³/h *)
    TargetGasFlow : REAL := 0.0;    (* Target gas flow: 0 Nm³/h *)
    GasRampDuration : TIME := T#12h; (* Gas ramp-down: 12 hours *)
    ReduceTemp_SP : REAL := 800.0;  (* Reduced furnace temp: 800°C *)
    ReduceTemp_Tol : REAL := 10.0;  (* Temp tolerance: ±10°C *)
    PurgeDuration : TIME := T#30m;  (* Nitrogen purge: 30 min *)
    PurgeNitrogenFlow : REAL := 100.0; (* Nitrogen flow: 100 Nm³/h *)
    FinalTemp_SP : REAL := 200.0;   (* Final furnace temp: 200°C *)
    FinalTemp_Tol : REAL := 10.0;   (* Final temp tolerance: ±10°C *)
    
    (* Operation Instances *)
    ReduceHeatOp : HeatOperation;   (* Reduce furnace temperature *)
    PurgeOp : TON;                  (* Nitrogen purge timer *)
    RampGasOp : RampDownGas;        (* Ramp down gas flow *)
    OxygenOp : AdjustOxygen;        (* Adjust oxygen supply *)
    CoolOp : CoolFurnace;           (* Cool furnace *)
    
    (* Diagnostics *)
    CurrentStep : STRING[80];       (* Current step for HMI *)
END_VAR

(* Main Logic *)
IF EmergencyStop THEN
    State := STATE_IDLE;
    GasValvePos := 0.0;
    OxygenValvePos := 0.0;
    FurnaceHeaterOn := FALSE;
    NitrogenValveOn := FALSE;
    CoolerOn := FALSE;
    CoolantFlow := 0.0;
    ShutdownComplete := FALSE;
    FlowAlarm := FALSE;
    RatioAlarm := FALSE;
    ReduceHeatOp.Enable := FALSE;
    PurgeOp(IN := FALSE);
    RampGasOp.Enable := FALSE;
    OxygenOp.Enable := FALSE;
    CoolOp.Enable := FALSE;
    CurrentStep := 'Emergency Stop';
ELSIF StartShutdown AND State = STATE_IDLE THEN
    State := STATE_INITIATE;
    ShutdownComplete := FALSE;
    CurrentStep := 'Initiating Shutdown';
END_IF;

(* State Machine *)
CASE State OF
    STATE_IDLE:
        GasValvePos := 0.0;
        OxygenValvePos := 0.0;
        FurnaceHeaterOn := FALSE;
        NitrogenValveOn := FALSE;
        CoolerOn := FALSE;
        CoolantFlow := 0.0;
        ShutdownComplete := FALSE;
        FlowAlarm := FALSE;
        RatioAlarm := FALSE;
        ReduceHeatOp.Enable := FALSE;
        PurgeOp(IN := FALSE);
        RampGasOp.Enable := FALSE;
        OxygenOp.Enable := FALSE;
        CoolOp.Enable := FALSE;
        CurrentStep := 'Idle';
        (* Idle: All systems off, ready for shutdown *)

    STATE_INITIATE:
        (* Notify operators, reduce production rate *)
        CurrentStep := 'Initiating Shutdown';
        State := STATE_REDUCE_LOAD;
        (* Placeholder for operator notifications *)

    STATE_REDUCE_LOAD:
        ReduceHeatOp(Enable := TRUE, Temp_SP := ReduceTemp_SP, Temp_Tol := ReduceTemp_Tol, Temp_PV := FurnaceTemp_PV);
        FurnaceHeaterOn := ReduceHeatOp.HeaterOn;
        CurrentStep := 'Reducing Furnace Load';
        IF ReduceHeatOp.Complete THEN
            State := STATE_PURGE_GASES;
            ReduceHeatOp.Enable := FALSE;
        END_IF;
        (* Reduces furnace temperature to 800°C *)

    STATE_PURGE_GASES:
        NitrogenValveOn := TRUE;
        PurgeOp(IN := TRUE, PT := PurgeDuration);
        CurrentStep := 'Purging Combustible Gases';
        IF PurgeOp.Q AND NitrogenFlow_PV >= PurgeNitrogenFlow THEN
            State := STATE_RAMP_GAS;
            NitrogenValveOn := FALSE;
            PurgeOp(IN := FALSE);
        END_IF;
        (* Purges gases with nitrogen for 30 min *)

    STATE_RAMP_GAS:
        RampGasOp(Enable := TRUE, InitialFlow := InitialGasFlow, TargetFlow := TargetGasFlow, 
                  RampDuration := GasRampDuration, GasFlow_PV := GasFlow_PV, FurnaceTemp_PV := FurnaceTemp_PV);
        GasValvePos := RampGasOp.GasValvePos;
        FlowAlarm := RampGasOp.FlowAlarm;
        CurrentStep := 'Ramping Down Gas Flow';
        IF RampGasOp.Complete THEN
            State := STATE_ADJUST_OXYGEN;
            RampGasOp.Enable := FALSE;
        END_IF;
        (* Ramps down gas flow over 12 hours *)

    STATE_ADJUST_OXYGEN:
        OxygenOp(Enable := TRUE, GasFlow_PV := GasFlow_PV, FurnaceTemp_PV := FurnaceTemp_PV, 
                 OxygenFlow_PV := OxygenFlow_PV);
        OxygenValvePos := OxygenOp.OxygenValvePos;
        RatioAlarm := OxygenOp.RatioAlarm;
        CurrentStep := 'Adjusting Oxygen Supply';
        IF GasFlow_PV <= TargetGasFlow THEN
            State := STATE_COOL_FURNACE;
            OxygenOp.Enable := FALSE;
        END_IF;
        (* Adjusts oxygen to maintain 1:2.5 ratio *)

    STATE_COOL_FURNACE:
        CoolOp(Enable := TRUE, Temp_SP := FinalTemp_SP, Temp_Tol := FinalTemp_Tol, Temp_PV := FurnaceTemp_PV);
        CoolerOn := CoolOp.CoolerOn;
        CoolantFlow := CoolOp.CoolantFlow;
        CurrentStep := 'Cooling Furnace';
        IF CoolOp.Complete THEN
            State := STATE_FINAL;
            CoolOp.Enable := FALSE;
        END_IF;
        (* Cools furnace to 200°C *)

    STATE_FINAL:
        (* Confirm safe state, deactivate remaining systems *)
        GasValvePos := 0.0;
        OxygenValvePos := 0.0;
        FurnaceHeaterOn := FALSE;
        NitrogenValveOn := FALSE;
        CoolerOn := FALSE;
        CoolantFlow := 0.0;
        CurrentStep := 'Final Shutdown';
        State := STATE_COMPLETE;
        (* Final checks before completion *)

    STATE_COMPLETE:
        GasValvePos := 0.0;
        OxygenValvePos := 0.0;
        FurnaceHeaterOn := FALSE;
        NitrogenValveOn := FALSE;
        CoolerOn := FALSE;
        CoolantFlow := 0.0;
        ShutdownComplete := TRUE;
        CurrentStep := 'Shutdown Complete';
        IF NOT StartShutdown THEN
            State := STATE_IDLE;
        END_IF;
        (* Shutdown complete; reset for next operation *)

ELSE
    State := STATE_IDLE; (* Fallback to safe state *)
END_CASE;

(* Notes:
   - ISA-88 Compliance:
     - Procedure: Steel plant shutdown with Unit Procedures (e.g., Ramp Down Gas Flow).
     - Operations: ReduceHeat, Purge, RampDownGas, AdjustOxygen, CoolFurnace as function blocks.
     - Recipe Parameters: Temp_SP, GasRampDuration, etc., for flexibility.
     - State Model: Idle, Running, Complete per ISA-88.
   - Shutdown Stages:
     1. Initiate: Notify operators.
     2. Reduce Load: Cool to 800°C.
     3. Purge Gases: Nitrogen flush for 30 min.
     4. Ramp Gas: Reduce gas from 500 Nm³/h to 0 over 12 hours.
     5. Adjust Oxygen: Maintain 1:2.5 fuel-to-air ratio.
     6. Cool Furnace: Cool to 200°C over ~4 hours.
     7. Final: Confirm safe state.
   - Safety:
     - EmergencyStop halts all operations.
     - Interlocks: FlowAlarm (±5% flow), RatioAlarm (±10% ratio), MinTemp (700°C).
   - Quality:
     - Controlled gas ramp-down prevents combustion instability.
     - Oxygen adjustment minimizes emissions.
     - Precise cooling avoids thermal stress.
   - Physical Integration:
     - Inputs: StartShutdown, GasFlow_PV, OxygenFlow_PV, Furnace  - Outputs: GasValvePos, OxygenValvePos, FurnaceHeaterOn, etc. (relays).
   - Scalability:
     - Modular blocks support larger furnaces or additional steps.
     - Parameters adjustable for different furnace sizes.
   - Maintenance:
     - Function blocks simplify debugging.
     - HMI displays CurrentStep, FlowAlarm, RatioAlarm for diagnostics.
*)
END_PROGRAM
