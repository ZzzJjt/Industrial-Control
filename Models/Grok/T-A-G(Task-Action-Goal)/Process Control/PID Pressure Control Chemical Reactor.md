(* Program: PID Feedback Control for Chemical Reactor Pressure *)
(* Version: 1.0, Date: 2025-05-21 *)
(* Maintains reactor pressure at 5.0 bar using PID control with 100 ms sampling *)
(* Adjusts pressure control valve with safety checks for real-time operation *)
PROGRAM PRG_PressureControl
VAR
    (* Inputs *)
    Pressure_PV : REAL;               (* Measured reactor pressure, bar *)
    EmergencyStop : BOOL;             (* TRUE if emergency stop activated *)
    
    (* Outputs *)
    Valve_Output : REAL;              (* Pressure control valve position, 0.0–100.0% *)
    PressureError : BOOL;             (* TRUE if pressure error exceeds threshold *)
    AlarmActive : BOOL;               (* TRUE if fault or emergency stop *)
    
    (* Internal Constants *)
    Pressure_SP : REAL := 5.0;        (* Pressure setpoint, 5.0 bar *)
    Kp : REAL := 2.0;                 (* Proportional gain *)
    Ki : REAL := 0.8;                 (* Integral gain *)
    Kd : REAL := 0.3;                 (* Derivative gain *)
    Valve_Min : REAL := 0.0;          (* Minimum valve position, % *)
    Valve_Max : REAL := 100.0;        (* Maximum valve position, % *)
    Error_Max : REAL := 2.0;          (* Maximum allowable error, bar *)
    Pressure_PV_Min : REAL := 0.0;    (* Minimum valid Pressure_PV, bar *)
    Pressure_PV_Max : REAL := 10.0;   (* Maximum valid Pressure_PV, bar *)
    Sample_Time : REAL := 0.1;        (* Sampling period, 100 ms *)
    
    (* Internal Variables *)
    Error : REAL;                     (* Current error: Pressure_SP - Pressure_PV *)
    Prev_Error : REAL;                (* Previous error for derivative *)
    Integral : REAL;                  (* Accumulated integral term *)
    Derivative : REAL;                (* Derivative term *)
    ValidPressure_PV : BOOL;          (* TRUE if Pressure_PV is valid *)
END_VAR

(* Initialize outputs and state *)
Valve_Output := 0.0;                  (* Valve closed *)
PressureError := FALSE;               (* No initial pressure error *)
AlarmActive := FALSE;                 (* No initial alarm *)
Prev_Error := 0.0;                    (* Initialize previous error *)
Integral := 0.0;                      (* Initialize integral term *)

(* Main logic *)
(* Emergency stop handling: Overrides all operations *)
IF EmergencyStop THEN
    (* Halt valve operation and activate alarm *)
    Valve_Output := 0.0;              (* Close valve *)
    PressureError := FALSE;           (* Clear pressure error *)
    AlarmActive := TRUE;              (* Activate alarm *)
    Integral := 0.0;                  (* Reset integral to prevent windup *)
    RETURN;
END_IF;

(* Validate sensor input *)
(* Check for numerical stability and valid range *)
ValidPressure_PV := Pressure_PV >= Pressure_PV_Min AND Pressure_PV <= Pressure_PV_Max AND ABS(Pressure_PV) <= 1.0E10;

IF NOT ValidPressure_PV THEN
    (* Invalid sensor reading: Stop valve operation *)
    Valve_Output := 0.0;              (* Close valve *)
    PressureError := TRUE;            (* Flag pressure error *)
    AlarmActive := TRUE;              (* Activate alarm *)
    Integral := 0.0;                  (* Reset integral to prevent windup *)
    RETURN;
ELSE
    PressureError := FALSE;           (* Clear pressure error *)
    AlarmActive := FALSE;             (* Clear alarm unless error persists *)
END_IF;

(* PID calculations *)
(* Calculate error *)
Error := Pressure_SP - Pressure_PV;   (* Error = Setpoint - Process Variable *)

(* Check for large error *)
IF ABS(Error) > Error_Max THEN
    (* Large error: Flag fault *)
    PressureError := TRUE;
    AlarmActive := TRUE;              (* Activate alarm *)
    Valve_Output := 0.0;              (* Close valve *)
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
Valve_Output := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

(* Clamp valve output within safe operational range *)
IF Valve_Output > Valve_Max THEN
    Valve_Output := Valve_Max;        (* Limit to maximum position *)
ELSIF Valve_Output < Valve_Min THEN
    Valve_Output := Valve_Min;        (* Limit to minimum position *)
END_IF;

(* Update previous error for next cycle *)
Prev_Error := Error;

(* Ensure safe state on power-up or PLC stop *)
(* Valve closed if fault or emergency *)
IF EmergencyStop OR PressureError THEN
    Valve_Output := 0.0;              (* Close valve *)
    Integral := 0.0;                  (* Reset integral *)
END_IF;

END_PROGRAM
