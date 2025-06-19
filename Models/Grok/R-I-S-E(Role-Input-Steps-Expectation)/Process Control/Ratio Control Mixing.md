VAR
    (* Inputs *)
    Flow_A_PV      : REAL;             (* Measured flow rate of reactant A [L/min] *)
    Flow_B_PV      : REAL;             (* Measured flow rate of reactant B [L/min] *)

    (* Control variables *)
    Flow_B_SP      : REAL;             (* Calculated flow setpoint for reactant B [L/min] *)
    Actual_Ratio   : REAL;             (* Current A:B flow ratio *)
    Error          : REAL;             (* Ratio error *)
    
    (* Parameters *)
    Ratio_Setpoint : REAL := 2.0;      (* Target A:B flow ratio *)
    Tolerance      : REAL := 0.05;     (* Acceptable ratio deviation *)
    
    (* Alarm *)
    Ratio_Alarm    : BOOL := FALSE;    (* Alarm for excessive ratio deviation *)
    
    (* Safety limits *)
    Max_Flow_B     : REAL := 100.0;    (* Maximum allowable flow for reactant B [L/min] *)
    Min_Flow_B     : REAL := 0.0;      (* Minimum allowable flow for reactant B [L/min] *)
END_VAR

(* Ratio control logic *)

(* Calculate actual A:B ratio *)
IF Flow_B_PV > 0.0 THEN
    Actual_Ratio := Flow_A_PV / Flow_B_PV;
ELSE
    Actual_Ratio := 0.0;  (* Avoid division by zero *)
END_IF;

(* Compute target flow setpoint for reactant B *)
Flow_B_SP := Flow_A_PV / Ratio_Setpoint;

(* Clamp Flow_B_SP to safe operational limits *)
IF Flow_B_SP > Max_Flow_B THEN
    Flow_B_SP := Max_Flow_B;
ELSIF Flow_B_SP < Min_Flow_B THEN
    Flow_B_SP := Min_Flow_B;
END_IF;

(* Evaluate ratio deviation and trigger alarm if needed *)
Error := Actual_Ratio - Ratio_Setpoint;
IF ABS(Error) > Tolerance THEN
    Ratio_Alarm := TRUE;  (* Trigger alarm for corrective action or operator notification *)
ELSE
    Ratio_Alarm := FALSE;
END_IF;

(* Flow_B_SP is sent to a flow control loop or actuator *)
(* Example: Use Flow_B_SP as setpoint for PID loop controlling reactant B pump *)
