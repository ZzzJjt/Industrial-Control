(* Program: PID Feedback Control for Distillation Column Level *)
(* Version: 1.0, Date: 2025-05-21 *)
(* Maintains liquid level at 60.0% using PID control with 100 ms sampling *)
(* Adjusts inlet valve position with safety checks for real-time operation *)
PROGRAM PRG_LevelControl
VAR
    (* Inputs *)
    Level_PV : REAL;                  (* Measured liquid level, % *)
    EmergencyStop : BOOL;             (* TRUE if emergency stop activated *)
    
    (* Outputs *)
    Valve_Position : REAL;            (* Inlet valve position, 0.0–100.0% *)
    LevelError : BOOL;                (* TRUE if level error exceeds threshold *)
    AlarmActive : BOOL;               (* TRUE if fault or emergency stop *)
    
    (* Internal Constants *)
    Level_SP : REAL := 60.0;          (* Level setpoint, 60.0% *)
    Kp : REAL := 1.5;                 (* Proportional gain *)
    Ki : REAL := 0.4;                 (* Integral gain *)
    Kd : REAL := 0.2;                 (* Derivative gain *)
    Valve_Min : REAL := 0.0;          (* Minimum valve position, % *)
    Valve_Max : REAL := 100.0;        (* Maximum valve position, % *)
    Error_Max : REAL := 20.0;         (* Maximum allowable error, % *)
    Level_PV_Min : REAL := 0.0;       (* Minimum valid Level_PV, % *)
    Level_PV_Max : REAL := 100.0;     (* Maximum valid Level_PV, % *)
    Sample_Time : REAL := 0.1;        (* Sampling period, 100 ms *)
    
    (* Internal Variables *)
    Error : REAL;                     (* Current error: Level_SP - Level_PV *)
    Prev_Error : REAL;                (* Previous error for derivative *)
    Integral : REAL;                  (* Accumulated integral term *)
    Derivative : REAL;                (* Derivative term *)
    ValidLevel_PV : BOOL;             (* TRUE if Level_PV is valid *)
END_VAR

(* Initialize outputs and state *)
Valve_Position := 0.0;                (* Valve closed *)
LevelError := FALSE;                  (* No initial level error *)
AlarmActive := FALSE;                 (* No initial alarm *)
Prev_Error := 0.0;                    (* Initialize previous error *)
Integral := 0.0;                      (* Initialize integral term *)

(* Main logic *)
(* Emergency stop handling: Overrides all operations *)
IF EmergencyStop THEN
    (* Halt valve operation and activate alarm *)
    Valve_Position := 0.0;            (* Close valve *)
    LevelError := FALSE;              (* Clear level error *)
    AlarmActive := TRUE;              (* Activate alarm *)
    Integral := 0.0;                  (* Reset integral to prevent windup *)
    RETURN;
END_IF;

(* Validate sensor input *)
(* Check for numerical stability and valid range *)
ValidLevel_PV := Level_PV >= Level_PV_Min AND Level_PV <= Level_PV_Max AND ABS(Level_PV) <= 1.0E10;

IF NOT ValidLevel_PV THEN
    (* Invalid sensor reading: Stop valve operation *)
    Valve_Position := 0.0;            (* Close valve *)
    LevelError := TRUE;               (* Flag level error *)
    AlarmActive := TRUE;              (* Activate alarm *)
    Integral := 0.0;                  (* Reset integral to prevent windup *)
    RETURN;
ELSE
    LevelError := FALSE;              (* Clear level error *)
    AlarmActive := FALSE;             (* Clear alarm unless error persists *)
END_IF;

(* PID calculations *)
(* Calculate error *)
Error := Level_SP - Level_PV;         (* Error = Setpoint - Process Variable *)

(* Check for large error *)
IF ABS(Error) > Error_Max THEN
    (* Large error: Flag fault *)
    LevelError := TRUE;
    AlarmActive := TRUE;              (* Activate alarm *)
    Valve_Position := 0.0;            (* Close valve *)
    Integral := 0.0;                  (* Reset integral to prevent windup *)
    RETURN;
END_IF;

(* Update integral term *)
Integral := Integral + Error * Sample_Time; (* Accumulate error over time *)

(* Prevent integral windup by clamping *)
IF Integral * Ki > Valve_Max THEN
    Integral := Valve_Max / Ki;
ELSIF Integral * Ki < Valve_Min THEN
    Integral := Valve_Min / Ki;
END_IF;

(* Calculate derivative term *)
Derivative := (Error - Prev_Error) / Sample_Time; (* Rate of error change *)

(* Compute PID output *)
Valve_Position := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

(* Clamp valve position within safe operational range *)
IF Valve_Position > Valve_Max THEN
    Valve_Position := Valve_Max;      (* Limit to maximum position *)
ELSIF Valve_Position < Valve_Min THEN
    Valve_Position := Valve_Min;      (* Limit to minimum position *)
END_IF;

(* Update previous error for next cycle *)
Prev_Error := Error;

(* Ensure safe state on power-up or PLC stop *)
(* Valve closed if fault or emergency *)
IF EmergencyStop OR LevelError THEN
    Valve_Position := 0.0;            (* Close valve *)
    Integral := 0.0;                  (* Reset integral *)
END_IF;

END_PROGRAM
