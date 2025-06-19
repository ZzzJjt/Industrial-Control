PROGRAM ReactorPressureControl
VAR
    (* Inputs *)
    Pressure_PV : REAL; (* Measured reactor pressure (bar) *)
    
    (* Setpoint *)
    Pressure_SP : REAL := 5.0; (* Target pressure setpoint (bar) *)
    
    (* PID tuning parameters *)
    Kp : REAL := 2.0; (* Proportional gain *)
    Ki : REAL := 0.8; (* Integral gain *)
    Kd : REAL := 0.3; (* Derivative gain *)
    
    (* PID variables *)
    Error : REAL; (* Current error: Pressure_SP - Pressure_PV *)
    Prev_Error : REAL := 0.0; (* Previous error for derivative calculation *)
    Integral : REAL := 0.0; (* Integral term accumulation *)
    Derivative : REAL; (* Derivative term *)
    Valve_Output : REAL; (* PID output: valve position (%) *)
    
    (* Safety limits *)
    Valve_Min : REAL := 0.0; (* Minimum valve position (%) *)
    Valve_Max : REAL := 100.0; (* Maximum valve position (%) *)
    Pressure_Min : REAL := 0.0; (* Minimum valid pressure (bar) *)
    Pressure_Max : REAL := 10.0; (* Maximum valid pressure (bar) *)
    
    (* Timing *)
    Cycle_Time : REAL := 0.1; (* Control cycle time: 100 ms *)
    
    (* Internal variables *)
    ErrorCode : INT := 0; (* 0: Success, 1: Invalid input *)
END_VAR

(* Initialize outputs *)
Valve_Output := Valve_Min; (* Default to closed valve for safety *)
ErrorCode := 0;

(* Validate inputs *)
IF NOT IS_VALID_REAL(Pressure_PV) OR 
   Pressure_PV < Pressure_Min OR Pressure_PV > Pressure_Max THEN
    (* Invalid or out-of-range pressure: close valve *)
    Valve_Output := Valve_Min;
    Integral := 0.0; (* Reset integral to prevent windup *)
    ErrorCode := 1;
    RETURN;
END_IF;

(* PID algorithm *)
(* Calculate error *)
Error := Pressure_SP - Pressure_PV;

(* Proportional term *)
Valve_Output := Kp * Error;

(* Integral term with anti-windup *)
IF Valve_Output >= Valve_Min AND Valve_Output <= Valve_Max THEN
    (* Update integral only if output is within limits *)
    Integral := Integral + Error * Cycle_Time;
    Valve_Output := Valve_Output + Ki * Integral;
END_IF;

(* Derivative term *)
Derivative := (Error - Prev_Error) / Cycle_Time;
Valve_Output := Valve_Output + Kd * Derivative;

(* Clamp output to safe range *)
IF Valve_Output > Valve_Max THEN
    Valve_Output := Valve_Max;
    (* Prevent integral windup: hold integral if output is saturated *)
    IF Error > 0.0 THEN
        Integral := Integral - Error * Cycle_Time;
    END_IF;
ELSIF Valve_Output < Valve_Min THEN
    Valve_Output := Valve_Min;
    (* Prevent integral windup *)
    IF Error < 0.0 THEN
        Integral := Integral - Error * Cycle_Time;
    END_IF;
END_IF;

(* Validate output *)
IF NOT IS_VALID_REAL(Valve_Output) THEN
    Valve_Output := Valve_Min;
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
