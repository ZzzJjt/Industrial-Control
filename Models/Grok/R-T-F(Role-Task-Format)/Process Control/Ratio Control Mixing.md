PROGRAM MixingRatioControl
VAR
    (* Inputs *)
    Flow_A_PV : REAL; (* Measured flow rate of Reactant A (L/min) *)
    Flow_B_PV : REAL; (* Measured flow rate of Reactant B (L/min) *)
    
    (* Parameters *)
    Ratio_Setpoint : REAL := 2.0; (* Desired A:B flow ratio (2:1) *)
    Tolerance : REAL := 0.05; (* Allowable deviation in ratio (e.g., 5%) *)
    
    (* Outputs *)
    Flow_B_SP : REAL; (* Setpoint for Reactant B flow (L/min) *)
    Actual_Ratio : REAL; (* Calculated A:B flow ratio *)
    Ratio_Error : REAL; (* Error between actual and desired ratio *)
    Ratio_Fault : BOOL := FALSE; (* TRUE if ratio deviates beyond tolerance *)
    
    (* Constraints *)
    Flow_A_Max : REAL := 100.0; (* Maximum expected flow for A (L/min) *)
    Flow_B_Max : REAL := 50.0; (* Maximum expected flow for B (L/min) *)
    Flow_B_SP_Min : REAL := 0.0; (* Minimum setpoint for B (L/min) *)
    Flow_B_SP_Max : REAL := 50.0; (* Maximum setpoint for B (L/min) *)
    
    (* Internal variables *)
    ErrorCode : INT := 0; (* 0: Success, 1: Invalid input *)
END_VAR

(* Initialize outputs *)
Flow_B_SP := Flow_B_SP_Min; (* Default to minimum setpoint for safety *)
Actual_Ratio := 0.0;
Ratio_Error := 0.0;
Ratio_Fault := FALSE;
ErrorCode := 0;

(* Validate inputs *)
IF NOT IS_VALID_REAL(Flow_A_PV) OR NOT IS_VALID_REAL(Flow_B_PV) OR
   Flow_A_PV < 0.0 OR Flow_A_PV > Flow_A_Max OR
   Flow_B_PV < 0.0 OR Flow_B_PV > Flow_B_Max THEN
    (* Invalid or out-of-range inputs: use minimum setpoint *)
    Flow_B_SP := Flow_B_SP_Min;
    Actual_Ratio := 0.0;
    Ratio_Error := 0.0;
    Ratio_Fault := TRUE; (* Flag fault for invalid inputs *)
    ErrorCode := 1;
    RETURN;
END_IF;

(* Calculate actual ratio (A:B) *)
IF Flow_B_PV > 0.001 THEN (* Avoid division by zero *)
    Actual_Ratio := Flow_A_PV / Flow_B_PV;
ELSE
    Actual_Ratio := 0.0; (* No flow in B: ratio undefined *)
END_IF;

(* Compute required flow setpoint for Reactant B *)
(* Flow_B_SP = Flow_A_PV / Ratio_Setpoint *)
Flow_B_SP := Flow_A_PV / Ratio_Setpoint;

(* Clamp setpoint within safe operational bounds *)
IF Flow_B_SP > Flow_B_SP_Max THEN
    Flow_B_SP := Flow_B_SP_Max;
ELSIF Flow_B_SP < Flow_B_SP_Min THEN
    Flow_B_SP := Flow_B_SP_Min;
END_IF;

(* Calculate ratio error for monitoring *)
Ratio_Error := Actual_Ratio - Ratio_Setpoint;

(* Check for ratio deviation *)
IF ABS(Ratio_Error) > Tolerance THEN
    Ratio_Fault := TRUE; (* Flag deviation beyond tolerance *)
ELSE
    Ratio_Fault := FALSE;
END_IF;

(* Validate output *)
IF NOT IS_VALID_REAL(Flow_B_SP) THEN
    Flow_B_SP := Flow_B_SP_Min;
    Ratio_Fault := TRUE;
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
