(* Program: PID Feedback Control for pH Regulation *)
(* Version: 1.0, Date: 2025-05-21 *)
(* Maintains process pH at 7.0 using PID control with 100 ms sampling *)
(* Adjusts acid/base dosing with safety checks for real-time operation *)
PROGRAM PRG_pHControl
VAR
    (* Inputs *)
    pH_PV : REAL;                     (* Measured pH value *)
    EmergencyStop : BOOL;             (* TRUE if emergency stop activated *)
    
    (* Outputs *)
    Dosing_Output : REAL;             (* Dosing pump/valve control signal, 0.0–100.0 *)
    pH_Error : BOOL;                  (* TRUE if pH error exceeds threshold *)
    AlarmActive : BOOL;               (* TRUE if fault or emergency stop *)
    
    (* Internal Constants *)
    pH_SP : REAL := 7.0;              (* pH setpoint, 7.0 *)
    Kp : REAL := 2.5;                 (* Proportional gain *)
    Ki : REAL := 0.6;                 (* Integral gain *)
    Kd : REAL := 0.3;                 (* Derivative gain *)
    Dosing_Min : REAL := 0.0;         (* Minimum dosing output *)
    Dosing_Max : REAL := 100.0;       (* Maximum dosing output *)
    Error_Max : REAL := 2.0;          (* Maximum allowable error, pH units *)
    pH_PV_Min : REAL := 0.0;          (* Minimum valid pH_PV *)
    pH_PV_Max : REAL := 14.0;         (* Maximum valid pH_PV *)
    Sample_Time : REAL := 0.1;        (* Sampling period, 100 ms *)
    
    (* Internal Variables *)
    Error : REAL;                     (* Current error: pH_SP - pH_PV *)
    Prev_Error : REAL;                (* Previous error for derivative *)
    Integral : REAL;                  (* Accumulated integral term *)
    Derivative : REAL;                (* Derivative term *)
    Valid_pH_PV : BOOL;               (* TRUE if pH_PV is valid *)
END_VAR

(* Initialize outputs and state *)
Dosing_Output := 0.0;                 (* Dosing off *)
pH_Error := FALSE;                    (* No initial pH error *)
AlarmActive := FALSE;                 (* No initial alarm *)
Prev_Error := 0.0;                    (* Initialize previous error *)
Integral := 0.0;                      (* Initialize integral term *)

(* Main logic *)
(* Emergency stop handling: Overrides all operations *)
IF EmergencyStop THEN
    (* Halt dosing and activate alarm *)
    Dosing_Output := 0.0;             (* Stop dosing *)
    pH_Error := FALSE;                (* Clear pH error *)
    AlarmActive := TRUE;              (* Activate alarm *)
    Integral := 0.0;                  (* Reset integral to prevent windup *)
    RETURN;
END_IF;

(* Validate sensor input *)
(* Check for numerical stability and valid range *)
Valid_pH_PV := pH_PV >= pH_PV_Min AND pH_PV <= pH_PV_Max AND ABS(pH_PV) <= 1.0E10;

IF NOT Valid_pH_PV THEN
    (* Invalid sensor reading: Stop dosing *)
    Dosing_Output := 0.0;             (* Stop dosing *)
    pH_Error := TRUE;                 (* Flag pH error *)
    AlarmActive := TRUE;              (* Activate alarm *)
    Integral := 0.0;                  (* Reset integral to prevent windup *)
    RETURN;
ELSE
    pH_Error := FALSE;                (* Clear pH error *)
    AlarmActive := FALSE;             (* Clear alarm unless error persists *)
END_IF;

(* PID calculations *)
(* Calculate error *)
Error := pH_SP - pH_PV;              (* Error = Setpoint - Process Variable *)

(* Check for large error *)
IF ABS(Error) > Error_Max THEN
    (* Large error: Flag fault *)
    pH_Error := TRUE;
    AlarmActive := TRUE;              (* Activate alarm *)
    Dosing_Output := 0.0;             (* Stop dosing *)
    Integral := 0.0;                  (* Reset integral to prevent windup *)
    RETURN;
END_IF;

(* Update integral term *)
Integral := Integral + Error * Sample_Time; (* Accumulate error over time *)

(* Prevent integral windup by clamping *)
IF Integral * Ki > Dosing_Max THEN
    Integral := Dosing_Max / Ki;
ELSIF Integral * Ki < Dosing_Min THEN
    Integral := Dosing_Min / Ki;
END_IF;

(* Calculate derivative term *)
Derivative := (Error - Prev_Error) / Sample_Time; (* Rate of error change *)

(* Compute PID output *)
Dosing_Output := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

(* Clamp dosing output within safe operational range *)
IF Dosing_Output > Dosing_Max THEN
    Dosing_Output := Dosing_Max;      (* Limit to maximum dosing *)
ELSIF Dosing_Output < Dosing_Min THEN
    Dosing_Output := Dosing_Min;      (* Limit to minimum dosing *)
END_IF;

(* Update previous error for next cycle *)
Prev_Error := Error;

(* Ensure safe state on power-up or PLC stop *)
(* Dosing stopped if fault or emergency *)
IF EmergencyStop OR pH_Error THEN
    Dosing_Output := 0.0;             (* Stop dosing *)
    Integral := 0.0;                  (* Reset integral *)
END_IF;

END_PROGRAM
