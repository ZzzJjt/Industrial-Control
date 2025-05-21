(* Program: PID Feedback Control for Gas Turbine Temperature *)
(* Version: 1.0, Date: 2025-05-21 *)
(* Maintains turbine temperature at 950.0°C using PID control with 100 ms sampling *)
(* Adjusts inlet valve position with safety checks for real-time operation *)
PROGRAM PRG_TurbineTempControl
VAR
    (* Inputs *)
    Temp_PV : REAL;                   (* Measured turbine temperature, °C *)
    EmergencyStop : BOOL;             (* TRUE if emergency stop activated *)
    
    (* Outputs *)
    Valve_Position : REAL;            (* Inlet valve position, 0.0–100.0% *)
    TempError : BOOL;                 (* TRUE if temperature error exceeds threshold *)
    AlarmActive : BOOL;               (* TRUE if fault or emergency stop *)
    
    (* Internal Constants *)
    Temp_SP : REAL := 950.0;          (* Temperature setpoint, 950.0°C *)
    Kp : REAL := 3.0;                 (* Proportional gain *)
    Ki : REAL := 0.7;                 (* Integral gain *)
    Kd : REAL := 0.2;                 (* Derivative gain *)
    Valve_Min : REAL := 0.0;          (* Minimum valve position, % *)
    Valve_Max : REAL := 100.0;        (* Maximum valve position, % *)
    Error_Max : REAL := 50.0;         (* Maximum allowable error, °C *)
    Temp_PV_Min : REAL := 0.0;        (* Minimum valid Temp_PV, °C *)
    Temp_PV_Max : REAL := 1200.0;     (* Maximum valid Temp_PV, °C *)
    Sample_Time : REAL := 0.1;        (* Sampling period, 100 ms *)
    
    (* Internal Variables *)
    Error : REAL;                     (* Current error: Temp_SP - Temp_PV *)
    Prev_Error : REAL;                (* Previous error for derivative *)
    Integral : REAL;                  (* Accumulated integral term *)
    Derivative : REAL;                (* Derivative term *)
    ValidTemp_PV : BOOL;              (* TRUE if Temp_PV is valid *)
END_VAR

(* Initialize outputs and state *)
Valve_Position := 0.0;                (* Valve closed *)
TempError := FALSE;                   (* No initial temperature error *)
AlarmActive := FALSE;                 (* No initial alarm *)
Prev_Error := 0.0;                    (* Initialize previous error *)
Integral := 0.0;                      (* Initialize integral term *)

(* Main logic *)
(* Emergency stop handling: Overrides all operations *)
IF EmergencyStop THEN
    (* Halt valve operation and activate alarm *)
    Valve_Position := 0.0;            (* Close valve *)
    TempError := FALSE;               (* Clear temperature error *)
    AlarmActive := TRUE;              (* Activate alarm *)
    Integral := 0.0;                  (* Reset integral to prevent windup *)
    RETURN;
END_IF;

(* Validate sensor input *)
(* Check for numerical stability and valid range *)
ValidTemp_PV := Temp_PV >= Temp_PV_Min AND Temp_PV <= Temp_PV_Max AND ABS(Temp_PV) <= 1.0E10;

IF NOT ValidTemp_PV THEN
    (* Invalid sensor reading: Stop valve operation *)
    Valve_Position := 0.0;            (* Close valve *)
    TempError := TRUE;                (* Flag temperature error *)
    AlarmActive := TRUE;              (* Activate alarm *)
    Integral := 0.0;                  (* Reset integral to prevent windup *)
    RETURN;
ELSE
    TempError := FALSE;               (* Clear temperature error *)
    AlarmActive := FALSE;             (* Clear alarm unless error persists *)
END_IF;

(* PID calculations *)
(* Calculate error *)
Error := Temp_SP - Temp_PV;           (* Error = Setpoint - Process Variable *)

(* Check for large error *)
IF ABS(Error) > Error_Max THEN
    (* Large error: Flag fault *)
    TempError := TRUE;
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

(* Clamp valve output within safe operational range *)
IF Valve_Position > Valve_Max THEN
    Valve_Position := Valve_Max;      (* Limit to maximum position *)
ELSIF Valve_Position < Valve_Min THEN
    Valve_Position := Valve_Min;      (* Limit to minimum position *)
END_IF;

(* Update previous error for next cycle *)
Prev_Error := Error;

(* Ensure safe state on power-up or PLC stop *)
(* Valve closed if fault or emergency *)
IF EmergencyStop OR TempError THEN
    Valve_Position := 0.0;            (* Close valve *)
    Integral := 0.0;                  (* Reset integral *)
END_IF;

END_PROGRAM
