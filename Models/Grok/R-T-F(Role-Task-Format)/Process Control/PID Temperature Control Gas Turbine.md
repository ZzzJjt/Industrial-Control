PROGRAM GasTurbineTempControl
VAR
    (* Inputs *)
    Temp_PV : REAL; (* Measured turbine inlet temperature (°C) *)
    
    (* Setpoint *)
    Temp_SP : REAL := 950.0; (* Target temperature setpoint (°C) *)
    
    (* PID tuning parameters *)
    Kp : REAL := 3.0; (* Proportional gain *)
    Ki : REAL := 0.7; (* Integral gain *)
    Kd : REAL := 0.2; (* Derivative gain *)
    
    (* PID variables *)
    Error : REAL; (* Current error: Temp_SP - Temp_PV *)
    Prev_Error : REAL := 0.0; (* Previous error for derivative calculation *)
    Integral : REAL := 0.0; (* Integral term accumulation *)
    Derivative : REAL; (* Derivative term *)
    Valve_Position : REAL; (* PID output: valve position (%) *)
    
    (* Safety limits *)
    Valve_Min : REAL := 0.0; (* Minimum valve position (%) *)
    Valve_Max : REAL := 100.0; (* Maximum valve position (%) *)
    Temp_Min : REAL := 0.0; (* Minimum valid temperature (°C) *)
    Temp_Max : REAL := 1200.0; (* Maximum valid temperature (°C) *)
    
    (* Timing *)
    Cycle_Time : REAL := 0.1; (* Control cycle time: 100 ms *)
    
    (* Internal variables *)
    ErrorCode : INT := 0; (* 0: Success, 1: Invalid input *)
END_VAR

(* Initialize outputs *)
Valve_Position := Valve_Min; (* Default to closed valve for safety *)
ErrorCode := 0;

(* Validate inputs *)
IF NOT IS_VALID_REAL(Temp_PV) OR 
   Temp_PV < Temp_Min OR Temp_PV > Temp_Max THEN
    (* Invalid or out-of-range temperature: close valve *)
    Valve_Position := Valve_Min;
    Integral := 0.0; (* Reset integral to prevent windup *)
    ErrorCode := 1;
    RETURN;
END_IF;

(* PID algorithm *)
(* Calculate error *)
Error := Temp_SP - Temp_PV;

(* Proportional term *)
Valve_Position := Kp * Error;

(* Integral term with anti-windup *)
IF Valve_Position >= Valve_Min AND Valve_Position <= Valve_Max THEN
    (* Update integral only if output is within limits *)
    Integral := Integral + Error * Cycle_Time;
    Valve_Position := Valve_Position + Ki * Integral;
END_IF;

(* Derivative term *)
Derivative := (Error - Prev_Error) / Cycle_Time;
Valve_Position := Valve_Position + Kd * Derivative;

(* Clamp output to safe range *)
IF Valve_Position > Valve_Max THEN
    Valve_Position := Valve_Max;
    (* Prevent integral windup: hold integral if output is saturated *)
    IF Error > 0.0 THEN
        Integral := Integral - Error * Cycle_Time;
    END_IF;
ELSIF Valve_Position < Valve_Min THEN
    Valve_Position := Valve_Min;
    (* Prevent integral windup *)
    IF Error < 0.0 THEN
        Integral := Integral - Error * Cycle_Time;
    END_IF;
END_IF;

(* Validate output *)
IF NOT IS_VALID_REAL(Valve_Position) THEN
    Valve_Position := Valve_Min;
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
