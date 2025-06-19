PROGRAM ChlorineDosingControl
VAR
    (* Inputs *)
    Dosing_PV : REAL; (* Measured chlorine concentration (ppm) *)
    FlowRate : REAL; (* Water flow rate (L/min, optional for future use) *)
    
    (* Setpoint *)
    Dosing_SP : REAL := 3.0; (* Target chlorine concentration (ppm) *)
    
    (* PID tuning parameters *)
    Kp : REAL := 2.0; (* Proportional gain *)
    Ki : REAL := 0.5; (* Integral gain *)
    Kd : REAL := 0.1; (* Derivative gain *)
    
    (* PID variables *)
    Error : REAL; (* Current error: Dosing_SP - Dosing_PV *)
    Prev_Error : REAL := 0.0; (* Previous error for derivative calculation *)
    Integral : REAL := 0.0; (* Integral term accumulation *)
    Derivative : REAL; (* Derivative term *)
    Dosing_Output : REAL; (* PID output: dosing rate (ppm) *)
    
    (* Safety limits *)
    Max_Dose : REAL := 10.0; (* Maximum dosing rate (ppm) *)
    Min_Dose : REAL := 0.0; (* Minimum dosing rate (ppm) *)
    
    (* Timing *)
    Cycle_Time : REAL := 0.1; (* Control cycle time: 100 ms *)
    
    (* Internal variables *)
    ErrorCode : INT := 0; (* 0: Success, 1: Invalid input *)
END_VAR

(* Initialize outputs *)
Dosing_Output := Min_Dose; (* Default to minimum dosing for safety *)
ErrorCode := 0;

(* Validate inputs *)
IF NOT IS_VALID_REAL(Dosing_PV) OR NOT IS_VALID_REAL(FlowRate) OR
   Dosing_PV < 0.0 OR FlowRate < 0.0 THEN
    (* Invalid or negative inputs: use minimum dosing *)
    Dosing_Output := Min_Dose;
    Integral := 0.0; (* Reset integral to prevent windup *)
    ErrorCode := 1;
    RETURN;
END_IF;

(* PID algorithm *)
(* Calculate error *)
Error := Dosing_SP - Dosing_PV;

(* Proportional term *)
Dosing_Output := Kp * Error;

(* Integral term with anti-windup *)
IF Dosing_Output >= Min_Dose AND Dosing_Output <= Max_Dose THEN
    (* Update integral only if output is within limits *)
    Integral := Integral + Error * Cycle_Time;
    Dosing_Output := Dosing_Output + Ki * Integral;
END_IF;

(* Derivative term *)
Derivative := (Error - Prev_Error) / Cycle_Time;
Dosing_Output := Dosing_Output + Kd * Derivative;

(* Clamp output to safe range *)
IF Dosing_Output > Max_Dose THEN
    Dosing_Output := Max_Dose;
    (* Prevent integral windup: hold integral if output is saturated *)
    IF Error > 0.0 THEN
        Integral := Integral - Error * Cycle_Time;
    END_IF;
ELSIF Dosing_Output < Min_Dose THEN
    Dosing_Output := Min_Dose;
    (* Prevent integral windup *)
    IF Error < 0.0 THEN
        Integral := Integral - Error * Cycle_Time;
    END_IF;
END_IF;

(* Validate output *)
IF NOT IS_VALID_REAL(Dosing_Output) THEN
    Dosing_Output := Min_Dose;
    Integral := 0.0; (* Reset integral *)
    ErrorCode := 1;
END_IF;

(* Update previous error *)
Prev_Error := Error;

(* Helper function to check if a REAL value is valid *)
FUNCTION IS_VALID_REAL : BOOL
VAR_INPUT
    Value : REAL;
END_VAR
IS_VALID_REAL := NOT (Value = 0.0 / 0.0) AND NOT (Value = 1.0 / 0.0);
END_FUNCTION

END_PROGRAM
