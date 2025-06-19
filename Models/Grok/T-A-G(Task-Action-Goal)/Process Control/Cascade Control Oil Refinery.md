(* Program: Cascade Control for Oil Refinery Vessel Pressure *)
(* Version: 1.0, Date: 2025-05-21 *)
(* Implements outer pressure loop and inner flow loop for stable regulation *)
(* Executes cyclically: inner loop 100 ms, outer loop 500 ms *)
PROGRAM PRG_PressureCascadeControl
VAR
    (* Inputs *)
    Pressure_PV : REAL;               (* Measured vessel pressure, bar *)
    Flow_PV : REAL;                   (* Measured oil inflow rate, L/min *)
    EmergencyStop : BOOL;             (* TRUE if emergency stop activated *)
    
    (* Outputs *)
    Flow_Output : REAL;               (* Valve/pump control signal, 0.0–100.0% *)
    PressureError : BOOL;             (* TRUE if pressure error exceeds threshold *)
    FlowError : BOOL;                 (* TRUE if flow error exceeds threshold *)
    AlarmActive : BOOL;               (* TRUE if fault or emergency stop *)
    
    (* Internal Constants *)
    Pressure_SP : REAL := 12.0;       (* Pressure setpoint, 12.0 bar *)
    Kp_Outer : REAL := 1.2;           (* Outer loop gain, L/min per bar *)
    Kp_Inner : REAL := 2.5;           (* Inner loop gain, % per L/min *)
    Pressure_Error_Max : REAL := 2.0; (* Max allowable pressure error, bar *)
    Flow_Error_Max : REAL := 10.0;    (* Max allowable flow error, L/min *)
    Flow_SP_Min : REAL := 0.0;        (* Min flow setpoint, L/min *)
    Flow_SP_Max : REAL := 100.0;      (* Max flow setpoint, L/min *)
    Flow_Output_Min : REAL := 0.0;    (* Min valve output, % *)
    Flow_Output_Max : REAL := 100.0;  (* Max valve output, % *)
    Outer_Loop_Period : UINT := 5;    (* Outer loop runs every 5 cycles, ~500 ms *)
    
    (* Internal Variables *)
    Pressure_Error : REAL;            (* Pressure error: Pressure_SP - Pressure_PV *)
    Flow_SP : REAL;                   (* Flow setpoint from outer loop *)
    Flow_Error : REAL;                (* Flow error: Flow_SP - Flow_PV *)
    ValidPressure : BOOL;             (* TRUE if Pressure_PV is numerically valid *)
    ValidFlow : BOOL;                 (* TRUE if Flow_PV is numerically valid *)
    CycleCounter : UINT;              (* Counts cycles for outer loop timing *)
END_VAR

(* Initialize outputs *)
Flow_Output := 0.0;                   (* Valve closed *)
PressureError := FALSE;               (* No initial pressure error *)
FlowError := FALSE;                   (* No initial flow error *)
AlarmActive := FALSE;                 (* No initial alarm *)
CycleCounter := 0;                    (* Initialize cycle counter *)

(* Main logic *)
(* Emergency stop handling: Overrides all operations *)
IF EmergencyStop THEN
    (* Halt system and activate alarm *)
    Flow_Output := 0.0;               (* Close valve/pump *)
    PressureError := FALSE;           (* Clear pressure error *)
    FlowError := FALSE;               (* Clear flow error *)
    AlarmActive := TRUE;              (* Activate alarm *)
    CycleCounter := 0;                (* Reset cycle counter *)
    RETURN;
END_IF;

(* Validate sensor inputs *)
(* Check for numerical stability to prevent overflow *)
ValidPressure := ABS(Pressure_PV) <= 1.0E10;
ValidFlow := ABS(Flow_PV) <= 1.0E10;

IF NOT ValidPressure OR NOT ValidFlow THEN
    (* Invalid sensor readings: Stop system *)
    Flow_Output := 0.0;               (* Close valve/pump *)
    PressureError := TRUE;            (* Flag pressure error *)
    FlowError := TRUE;                (* Flag flow error *)
    AlarmActive := TRUE;              (* Activate alarm *)
    CycleCounter := 0;                (* Reset cycle counter *)
    RETURN;
END_IF;

(* Increment cycle counter for outer loop timing *)
CycleCounter := CycleCounter + 1;
IF CycleCounter >= Outer_Loop_Period THEN
    CycleCounter := 0;                (* Reset counter every 5 cycles *)
END_IF;

(* Outer loop: Pressure control *)
(* Runs every Outer_Loop_Period cycles (~500 ms) *)
IF CycleCounter = 0 THEN
    (* Calculate pressure error *)
    Pressure_Error := Pressure_SP - Pressure_PV; (* Error = Setpoint - PV *)
    
    (* Check for large pressure error *)
    IF ABS(Pressure_Error) > Pressure_Error_Max THEN
        (* Large error: Flag fault *)
        PressureError := TRUE;
        Flow_Output := 0.0;           (* Close valve/pump *)
        AlarmActive := TRUE;          (* Activate alarm *)
        FlowError := FALSE;           (* Clear flow error *)
        RETURN;
    ELSE
        PressureError := FALSE;       (* Clear pressure error *)
    END_IF;
    
    (* Compute flow setpoint using proportional control *)
    Flow_SP := Kp_Outer * Pressure_Error; (* Flow_SP = Kp_Outer * Error *)
    
    (* Limit flow setpoint to safe range *)
    IF Flow_SP < Flow_SP_Min THEN
        Flow_SP := Flow_SP_Min;
    ELSIF Flow_SP > Flow_SP_Max THEN
        Flow_SP := Flow_SP_Max;
    END_IF;
END_IF;

(* Inner loop: Flow control *)
(* Runs every cycle (~100 ms) *)
(* Calculate flow error *)
Flow_Error := Flow_SP - Flow_PV;      (* Error = Setpoint - PV *)

(* Check for large flow error *)
IF ABS(Flow_Error) > Flow_Error_Max THEN
    (* Large error: Flag fault *)
    FlowError := TRUE;
    AlarmActive := TRUE;              (* Activate alarm *)
ELSE
    FlowError := FALSE;               (* Clear flow error *)
END_IF;

(* Compute valve control signal using proportional control *)
Flow_Output := Kp_Inner * Flow_Error; (* Flow_Output = Kp_Inner * Error *)

(* Limit valve output to safe range *)
IF Flow_Output < Flow_Output_Min THEN
    Flow_Output := Flow_Output_Min;
ELSIF Flow_Output > Flow_Output_Max THEN
    Flow_Output := Flow_Output_Max;
END_IF;

(* Ensure safe state on power-up or PLC stop *)
(* Valve closed if fault or emergency *)
IF EmergencyStop OR PressureError OR FlowError THEN
    Flow_Output := 0.0;               (* Close valve/pump *)
END_IF;

END_PROGRAM
