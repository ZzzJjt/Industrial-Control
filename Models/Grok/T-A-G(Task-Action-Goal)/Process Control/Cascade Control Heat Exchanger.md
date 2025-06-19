(* Program: Cascade Control for Heat Exchanger *)
(* Version: 1.0, Date: 2025-05-21 *)
(* Implements outer temperature loop and inner flow loop for stable regulation *)
(* Executes cyclically (e.g., 100 ms) with safety checks *)
PROGRAM PRG_CascadeControl
VAR
    (* Inputs *)
    Temp_PV : REAL;                   (* Measured temperature, °C *)
    Flow_PV : REAL;                   (* Measured flow rate, L/min *)
    EmergencyStop : BOOL;             (* TRUE if emergency stop activated *)
    
    (* Outputs *)
    Flow_Output : REAL;               (* Valve control signal, 0.0–100.0% *)
    TempError : BOOL;                 (* TRUE if temperature error exceeds threshold *)
    FlowError : BOOL;                 (* TRUE if flow error exceeds threshold *)
    AlarmActive : BOOL;               (* TRUE if fault or emergency stop *)
    
    (* Internal Constants *)
    Temp_SP : REAL := 85.0;           (* Temperature setpoint, 85.0°C *)
    Kp_Outer : REAL := 1.0;           (* Outer loop proportional gain, L/min per °C *)
    Kp_Inner : REAL := 2.0;           (* Inner loop proportional gain, % per L/min *)
    Temp_Error_Max : REAL := 10.0;    (* Max allowable temperature error, °C *)
    Flow_Error_Max : REAL := 5.0;     (* Max allowable flow error, L/min *)
    Flow_SP_Min : REAL := 0.0;        (* Min flow setpoint, L/min *)
    Flow_SP_Max : REAL := 100.0;      (* Max flow setpoint, L/min *)
    Flow_Output_Min : REAL := 0.0;    (* Min valve output, % *)
    Flow_Output_Max : REAL := 100.0;  (* Max valve output, % *)
    
    (* Internal Variables *)
    Temp_Error : REAL;                (* Temperature error: Temp_SP - Temp_PV *)
    Flow_SP : REAL;                   (* Flow setpoint from outer loop *)
    Flow_Error : REAL;                (* Flow error: Flow_SP - Flow_PV *)
    ValidTemp : BOOL;                 (* TRUE if Temp_PV is numerically valid *)
    ValidFlow : BOOL;                 (* TRUE if Flow_PV is numerically valid *)
END_VAR

(* Initialize outputs *)
Flow_Output := 0.0;                   (* Valve closed *)
TempError := FALSE;                   (* No initial temperature error *)
FlowError := FALSE;                   (* No initial flow error *)
AlarmActive := FALSE;                 (* No initial alarm *)

(* Main logic *)
(* Emergency stop handling: Overrides all operations *)
IF EmergencyStop THEN
    (* Halt system and activate alarm *)
    Flow_Output := 0.0;               (* Close valve *)
    TempError := FALSE;               (* Clear temperature error *)
    FlowError := FALSE;               (* Clear flow error *)
    AlarmActive := TRUE;              (* Activate alarm *)
    RETURN;
END_IF;

(* Validate sensor inputs *)
(* Check for numerical stability to prevent overflow *)
ValidTemp := ABS(Temp_PV) <= 1.0E10;
ValidFlow := ABS(Flow_PV) <= 1.0E10;

IF NOT ValidTemp OR NOT ValidFlow THEN
    (* Invalid sensor readings: Stop system *)
    Flow_Output := 0.0;               (* Close valve *)
    TempError := TRUE;                (* Flag temperature error *)
    FlowError := TRUE;                (* Flag flow error *)
    AlarmActive := TRUE;              (* Activate alarm *)
    RETURN;
END_IF;

(* Outer loop: Temperature control *)
(* Calculate temperature error and generate flow setpoint *)
Temp_Error := Temp_SP - Temp_PV;      (* Error = Setpoint - Process Variable *)
IF ABS(Temp_Error) > Temp_Error_Max THEN
    (* Large temperature error: Flag fault *)
    TempError := TRUE;
    Flow_Output := 0.0;               (* Close valve *)
    AlarmActive := TRUE;              (* Activate alarm *)
    FlowError := FALSE;               (* Clear flow error *)
    RETURN;
ELSE
    TempError := FALSE;               (* Clear temperature error *)
END_IF;

(* Compute flow setpoint using proportional control *)
Flow_SP := Kp_Outer * Temp_Error;     (* Flow_SP = Kp_Outer * Temp_Error *)

(* Limit flow setpoint to safe range *)
IF Flow_SP < Flow_SP_Min THEN
    Flow_SP := Flow_SP_Min;
ELSIF Flow_SP > Flow_SP_Max THEN
    Flow_SP := Flow_SP_Max;
END_IF;

(* Inner loop: Flow control *)
(* Calculate flow error and generate valve control signal *)
Flow_Error := Flow_SP - Flow_PV;      (* Error = Setpoint - Process Variable *)
IF ABS(Flow_Error) > Flow_Error_Max THEN
    (* Large flow error: Flag fault *)
    FlowError := TRUE;
    AlarmActive := TRUE;              (* Activate alarm *)
ELSE
    FlowError := FALSE;               (* Clear flow error *)
END_IF;

(* Compute valve control signal using proportional control *)
Flow_Output := Kp_Inner * Flow_Error; (* Flow_Output = Kp_Inner * Flow_Error *)

(* Limit valve output to safe range *)
IF Flow_Output < Flow_Output_Min THEN
    Flow_Output := Flow_Output_Min;
ELSIF Flow_Output > Flow_Output_Max THEN
    Flow_Output := Flow_Output_Max;
END_IF;

(* Ensure safe state on power-up or PLC stop *)
(* Valve closed if fault or emergency *)
IF EmergencyStop OR TempError OR FlowError THEN
    Flow_Output := 0.0;               (* Close valve *)
END_IF;

END_PROGRAM
