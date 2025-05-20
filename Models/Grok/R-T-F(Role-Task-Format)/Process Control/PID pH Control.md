PROGRAM ChemicalPHControl
VAR
    (* Inputs *)
    pH_PV : REAL; (* Measured pH value *)
    
    (* Setpoint *)
    pH_SP : REAL := 7.0; (* Target pH setpoint *)
    
    (* PID tuning parameters *)
    Kp : REAL := 2.5; (* Proportional gain *)
    Ki : REAL := 0.6; (* Integral gain *)
    Kd : REAL := 0.3; (* Derivative gain *)
    
    (* PID variables *)
    Error : REAL; (* Current error: pH_SP - pH_PV, scaled for nonlinearity *)
    Prev_Error : REAL := 0.0; (* Previous error for derivative calculation *)
    Integral : REAL := 0.0; (* Integral term accumulation *)
    Derivative : REAL; (* Derivative term *)
    Dosing_Output : REAL; (* PID output: dosing rate (%) *)
    
    (* Safety limits *)
    Dosing_Min : REAL := 0.0; (* Minimum dosing rate (%) *)
    Dosing_Max : REAL := 100.0; (* Maximum dosing rate (%) *)
    pH_Min : REAL := 0.0; (* Minimum valid pH *)
    pH_Max : REAL := 14.0; (* Maximum valid pH *)
    
    (* Timing *)
    Cycle_Time : REAL := 0.1; (* Control cycle time: 100 ms *)
    
    (* Internal variables *)
    ErrorCode : INT := 0; (* 0: Success, 1: Invalid input *)
    Raw_Error : REAL; (* Unscaled error: pH_SP - pH_PV *)
END_VAR

(* Initialize outputs *)
Dosing_Output := Dosing_Min; (* Default to no dosing for safety *)
ErrorCode := 0;

(* Validate inputs *)
IF NOT IS_VALID_REAL(pH_PV) OR pH_PV < pH_Min OR pH_PV > pH_Max THEN
    (* Invalid or out-of-range pH: stop dosing *)
    Dosing_Output := Dosing_Min;
    Integral := 0.0; (* Reset integral to prevent windup *)
    ErrorCode := 1;
    RETURN;
END_IF;

(* PID algorithm *)
(* Calculate raw error *)
Raw_Error := pH_SP - pH_PV;

(* Scale error to account for pH's logarithmic nature *)
(* Use exponential scaling to amplify small errors near setpoint *)
Error := Raw_Error * (1.0 + 0.5 * EXP(-ABS(Raw_Error))); 

(* Proportional term *)
Dosing_Output := Kp * Error;

(* Integral term with anti-windup *)
IF Dosing_Output >= Dosing_Min AND Dosing_Output <= Dosing_Max THEN
    (* Update integral only if output is within limits *)
    Integral := Integral + Error * Cycle_Time;
    Dosing_Output := Dosing_Output + Ki * Integral;
END_IF;

(* Derivative term *)
Derivative := (Error - Prev_Error) / Cycle_Time;
Dosing_Output := Dosing_Output + Kd * Derivative;

(* Clamp output to safe range *)
IF Dosing_Output > Dosing_Max THEN
    Dosing_Output := Dosing_Max;
    (* Prevent integral windup: hold integral if output is saturated *)
    IF Error > 0.0 THEN
        Integral := Integral - Error * Cycle_Time;
    END_IF;
ELSIF Dosing_Output < Dosing_Min THEN
    Dosing_Output := Dosing_Min;
    (* Prevent integral windup *)
    IF Error < 0.0 THEN
        Integral := Integral - Error * Cycle_Time;
    END_IF;
END_IF;

(* Validate output *)
IF NOT IS_VALID_REAL(Dosing_Output) THEN
    Dosing_Output := Dosing_Min;
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
