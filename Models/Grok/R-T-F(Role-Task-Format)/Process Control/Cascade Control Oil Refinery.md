PROGRAM OilRefineryPressureControl
VAR
    (* Inputs *)
    Pressure_PV : REAL; (* Process variable: vessel pressure (bar) *)
    Flow_PV : REAL; (* Process variable: oil inflow rate (L/min) *)
    
    (* Setpoints *)
    Pressure_SP : REAL := 12.0; (* Pressure setpoint (bar) *)
    Flow_SP : REAL; (* Flow setpoint (L/min), set by outer loop *)
    
    (* Control errors *)
    Pressure_Error : REAL; (* Pressure error: Pressure_SP - Pressure_PV *)
    Flow_Error : REAL; (* Flow error: Flow_SP - Flow_PV *)
    
    (* Outputs *)
    Flow_Output : REAL; (* Control output to actuator (0 to 100%) *)
    
    (* Tuning parameters *)
    Kp_Outer : REAL := 1.2; (* Proportional gain for outer (pressure) loop *)
    Kp_Inner : REAL := 2.5; (* Proportional gain for inner (flow) loop *)
    
    (* Constraints *)
    Flow_SP_Min : REAL := 0.0; (* Minimum flow setpoint (L/min) *)
    Flow_SP_Max : REAL := 200.0; (* Maximum flow setpoint (L/min) *)
    Flow_Output_Min : REAL := 0.0; (* Minimum actuator position (%) *)
    Flow_Output_Max : REAL := 100.0; (* Maximum actuator position (%) *)
    
    (* Internal variables *)
    ErrorCode : INT := 0; (* 0: Success, 1: Invalid input *)
END_VAR

(* Initialize outputs *)
Flow_Output := 0.0;
ErrorCode := 0;

(* Validate inputs *)
IF NOT IS_VALID_REAL(Pressure_PV) OR NOT IS_VALID_REAL(Flow_PV) THEN
    (* Non-finite input (NaN or infinity) *)
    Flow_Output := Flow_Output_Min; (* Close actuator for safety *)
    ErrorCode := 1;
    RETURN;
END_IF;

(* Outer loop: Pressure control *)
(* Generates Flow_SP to correct pressure deviations *)
Pressure_Error := Pressure_SP - Pressure_PV; (* Calculate pressure error *)
Flow_SP := Kp_Outer * Pressure_Error; (* Proportional control for flow setpoint *)

(* Apply constraints to Flow_SP *)
IF Flow_SP < Flow_SP_Min THEN
    Flow_SP := Flow_SP_Min;
ELSIF Flow_SP > Flow_SP_Max THEN
    Flow_SP := Flow_SP_Max;
END_IF;

(* Inner loop: Flow control *)
(* Responds quickly to flow disturbances *)
Flow_Error := Flow_SP - Flow_PV; (* Calculate flow error *)
Flow_Output := Kp_Inner * Flow_Error; (* Proportional control for actuator *)

(* Apply constraints to Flow_Output *)
IF Flow_Output < Flow_Output_Min THEN
    Flow_Output := Flow_Output_Min;
ELSIF Flow_Output > Flow_Output_Max THEN
    Flow_Output := Flow_Output_Max;
END_IF;

(* Validate output *)
IF NOT IS_VALID_REAL(Flow_Output) THEN
    Flow_Output := Flow_Output_Min; (* Close actuator for safety *)
    ErrorCode := 1;
END_IF;

(* Helper function to check if a REAL value is valid *)
FUNCTION IS_VALID_REAL : BOOL
VAR_INPUT
    Value : REAL;
END_VAR
IS_VALID_REAL := NOT (Value = 0.0 / 0.0) AND NOT (Value = 1.0 / 0.0);
END_FUNCTION

END_PROGRAM
