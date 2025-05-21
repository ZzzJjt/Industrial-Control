(* Program: PID Feedback Control for Chlorine Dosing *)
(* Version: 1.0, Date: 2025-05-21 *)
(* Regulates chlorine concentration at 3.0 ppm with 100 ms sampling *)
(* Uses PID control with safety limits and real-time execution *)
PROGRAM PRG_ChlorineDosingControl
VAR
    (* Inputs *)
    Dosing_PV : REAL;                 (* Measured chlorine concentration, ppm *)
    FlowRate : REAL;                  (* Water flow rate, L/min, reserved for future use *)
    EmergencyStop : BOOL;             (* TRUE if emergency stop activated *)
    
    (* Outputs *)
    Dosing_Output : REAL;             (* Dosing pump control signal, 0.0–10.0 units *)
    DosingError : BOOL;               (* TRUE if dosing error exceeds threshold *)
    AlarmActive : BOOL;               (* TRUE if fault or emergency stop *)
    
    (* Internal Constants *)
    Dosing_SP : REAL := 3.0;          (* Chlorine setpoint, 3.0 ppm *)
    Kp : REAL := 2.0;                 (* Proportional gain *)
    Ki : REAL := 0.5;                 (* Integral gain *)
    Kd : REAL := 0.1;                 (* Derivative gain *)
    Min_Dose : REAL := 0.0;           (* Minimum dosing output, units *)
    Max_Dose : REAL := 10.0;          (* Maximum dosing output, units *)
    Error_Max : REAL := 2.0;          (* Maximum allowable error, ppm *)
    Sample_Time : REAL := 0.1;        (* Sampling period, 100 ms *)
    Dosing_PV_Min : REAL := 0.0;      (* Minimum valid Dosing_PV, ppm *)
    Dosing_PV_Max : REAL := 10.0;     (* Maximum valid Dosing_PV, ppm *)
    
    (* Internal Variables *)
    Error : REAL;                     (* Current error: Dosing_SP - Dosing_PV *)
    Prev_Error : REAL;                (* Previous error for derivative *)
    Integral : REAL;                  (* Accumulated integral term *)
    Derivative : REAL;                (* Derivative term *)
    ValidDosing_PV : BOOL;            (* TRUE if Dosing_PV is valid *)
END_VAR

(* Initialize outputs and state *)
Dosing_Output := 0.0;                 (* Dosing pump off *)
DosingError := FALSE;                 (* No initial dosing error *)
AlarmActive := FALSE;                 (* No initial alarm *)
Prev_Error := 0.0;                    (* Initialize previous error *)
Integral := 0.0;                      (* Initialize integral term *)

(* Main logic *)
(* Emergency stop handling: Overrides all operations *)
IF EmergencyStop THEN
    (* Halt dosing and activate alarm *)
    Dosing_Output := 0.0;             (* Stop dosing pump *)
    DosingError := FALSE;             (* Clear dosing error *)
    AlarmActive := TRUE;              (* Activate alarm *)
    Integral := 0.0;                  (* Reset integral to prevent windup *)
    RETURN;
END_IF;

(* Validate sensor input *)
(* Check for numerical stability and valid range *)
ValidDosing_PV := Dosing_PV >= Dosing_PV_Min AND Dosing_PV <= Dosing_PV_Max AND ABS(Dosing_PV) <= 1.0E10;

IF NOT ValidDosing_PV THEN
    (* Invalid sensor reading: Stop dosing *)
    Dosing_Output := 0.0;             (* Stop dosing pump *)
    DosingError := TRUE;              (* Flag dosing error *)
    AlarmActive := TRUE;              (* Activate alarm *)
    Integral := 0.0;                  (* Reset integral to prevent windup *)
    RETURN;
ELSE
    DosingError := FALSE;             (* Clear dosing error *)
    AlarmActive := FALSE;             (* Clear alarm unless error persists *)
END_IF;

(* PID calculations *)
(* Calculate error *)
Error := Dosing_SP - Dosing_PV;       (* Error = Setpoint - Process Variable *)

(* Check for large error *)
IF ABS(Error) > Error_Max THEN
    (* Large error: Flag fault *)
    DosingError := TRUE;
    AlarmActive := TRUE;              (* Activate alarm *)
    Dosing_Output := 0.0;             (* Stop dosing pump *)
    Integral := 0.0;                  (* Reset integral to prevent windup *)
    RETURN;
END_IF;

(* Update integral term *)
Integral := Integral + Error * Sample_Time; (* Accumulate error over time *)

(* Prevent integral windup by clamping *)
IF Integral * Ki > Max_Dose THEN
    Integral := Max_Dose / Ki;
ELSIF Integral * Ki < Min_Dose THEN
    Integral := Min_Dose / Ki;
END_IF;

(* Calculate derivative term *)
Derivative := (Error - Prev_Error) / Sample_Time; (* Rate of error change *)

(* Compute PID output *)
Dosing_Output := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

(* Clamp output within safe operational range *)
IF Dosing_Output > Max_Dose THEN
    Dosing_Output := Max_Dose;        (* Limit to maximum dose *)
ELSIF Dosing_Output < Min_Dose THEN
    Dosing_Output := Min_Dose;        (* Limit to minimum dose *)
END_IF;

(* Update previous error for next cycle *)
Prev_Error := Error;

(* Ensure safe state on power-up or PLC stop *)
(* Dosing stopped if fault or emergency *)
IF EmergencyStop OR DosingError THEN
    Dosing_Output := 0.0;             (* Stop dosing pump *)
    Integral := 0.0;                  (* Reset integral *)
END_IF;

END_PROGRAM
