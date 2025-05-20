PROGRAM ChemicalMixingControl
VAR
    (* Inputs *)
    Flow_B : REAL; (* Measured flow rate of Reactant B (L/min) *)
    Concentration_B : REAL := 0.8; (* Concentration of Reactant B (fraction, 0.0 to 1.0) *)
    Temperature_B : REAL := 25.0; (* Temperature of Reactant B (°C) *)
    
    (* Parameters *)
    Desired_Ratio : REAL := 2.0; (* Target A:B flow ratio *)
    Compensation_Factor : REAL := 1.0; (* Dynamic adjustment based on conditions *)
    
    (* Constraints *)
    Flow_A_Min : REAL := 0.0; (* Minimum flow setpoint for Reactant A (L/min) *)
    Flow_A_Max : REAL := 100.0; (* Maximum flow setpoint for Reactant A (L/min) *)
    Flow_B_Max : REAL := 50.0; (* Maximum expected Flow_B (L/min) *)
    Concentration_Min : REAL := 0.1; (* Minimum valid concentration *)
    Concentration_Max : REAL := 1.0; (* Maximum valid concentration *)
    Temperature_Min : REAL := 0.0; (* Minimum valid temperature (°C) *)
    Temperature_Max : REAL := 100.0; (* Maximum valid temperature (°C) *)
    
    (* Outputs *)
    Flow_A_Setpoint : REAL; (* Setpoint for Reactant A flow (L/min) *)
    
    (* Internal variables *)
    ErrorCode : INT := 0; (* 0: Success, 1: Invalid input *)
END_VAR

(* Initialize outputs *)
Flow_A_Setpoint := Flow_A_Min; (* Default to minimum flow for safety *)
ErrorCode := 0;

(* Validate inputs *)
IF NOT IS_VALID_REAL(Flow_B) OR NOT IS_VALID_REAL(Concentration_B) OR 
   NOT IS_VALID_REAL(Temperature_B) OR
   Flow_B < 0.0 OR Flow_B > Flow_B_Max OR
   Concentration_B < Concentration_Min OR Concentration_B > Concentration_Max OR
   Temperature_B < Temperature_Min OR Temperature_B > Temperature_Max THEN
    (* Invalid or out-of-range inputs: use minimum setpoint *)
    Flow_A_Setpoint := Flow_A_Min;
    ErrorCode := 1;
    RETURN;
END_IF;

(* Dynamic compensation factor *)
(* Adjust based on Concentration_B and Temperature_B *)
(* Example: Increase flow if concentration is low or temperature is high *)
Compensation_Factor := 1.0 * (0.8 / Concentration_B); (* Inverse concentration scaling *)
IF Temperature_B > 50.0 THEN
    Compensation_Factor := Compensation_Factor * 1.1; (* 10% increase for high temperature *)
END_IF;

(* Ensure Compensation_Factor is reasonable *)
IF Compensation_Factor < 0.5 THEN
    Compensation_Factor := 0.5;
ELSIF Compensation_Factor > 2.0 THEN
    Compensation_Factor := 2.0;
END_IF;

(* Feedforward calculation *)
(* Flow_A_Setpoint = Desired_Ratio * Flow_B * Compensation_Factor *)
Flow_A_Setpoint := Desired_Ratio * Flow_B * Compensation_Factor;

(* Clamp setpoint within safe operational bounds *)
IF Flow_A_Setpoint > Flow_A_Max THEN
    Flow_A_Setpoint := Flow_A_Max;
ELSIF Flow_A_Setpoint < Flow_A_Min THEN
    Flow_A_Setpoint := Flow_A_Min;
END_IF;

(* Validate output *)
IF NOT IS_VALID_REAL(Flow_A_Setpoint) THEN
    Flow_A_Setpoint := Flow_A_Min;
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
