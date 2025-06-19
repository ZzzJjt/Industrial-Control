(* Program: Ratio Control for Reactant Mixing *)
(* Version: 1.0, Date: 2025-05-21 *)
(* Maintains 2:1 flow ratio of Reactant A to Reactant B *)
(* Executes cyclically (e.g., 100 ms) with safety checks *)
PROGRAM PRG_RatioControl
VAR
    (* Inputs *)
    Flow_A_PV : REAL;                 (* Measured flow of Reactant A, L/min *)
    Flow_B_PV : REAL;                 (* Measured flow of Reactant B, L/min *)
    EmergencyStop : BOOL;             (* TRUE if emergency stop activated *)
    
    (* Outputs *)
    Flow_B_SP : REAL;                 (* Flow setpoint for Reactant B, L/min *)
    Ratio_Error : BOOL;               (* TRUE if ratio deviation exceeds tolerance *)
    AlarmActive : BOOL;               (* TRUE if fault or emergency stop *)
    
    (* Internal Constants *)
    Ratio_Setpoint : REAL := 2.0;     (* Desired A:B flow ratio, 2:1 *)
    Tolerance : REAL := 0.05;         (* Allowed ratio deviation *)
    Flow_A_Min : REAL := 0.0;         (* Minimum valid Flow_A_PV, L/min *)
    Flow_A_Max : REAL := 100.0;       (* Maximum valid Flow_A_PV, L/min *)
    Flow_B_Min : REAL := 0.0;         (* Minimum valid Flow_B_PV, L/min *)
    Flow_B_Max : REAL := 50.0;        (* Maximum valid Flow_B_PV, L/min *)
    Flow_B_SP_Min : REAL := 0.0;      (* Minimum valid Flow_B_SP, L/min *)
    Flow_B_SP_Max : REAL := 50.0;     (* Maximum valid Flow_B_SP, L/min *)
    
    (* Internal Variables *)
    Actual_Ratio : REAL;              (* Current A:B flow ratio *)
    Ratio_Deviation : REAL;           (* Deviation from Ratio_Setpoint *)
    Valid_Flow_A : BOOL;              (* TRUE if Flow_A_PV is valid *)
    Valid_Flow_B : BOOL;              (* TRUE if Flow_B_PV is valid *)
END_VAR

(* Initialize outputs *)
Flow_B_SP := 0.0;                     (* No flow setpoint initially *)
Ratio_Error := FALSE;                 (* No initial ratio error *)
AlarmActive := FALSE;                 (* No initial alarm *)

(* Main logic *)
(* Emergency stop handling: Overrides all operations *)
IF EmergencyStop THEN
    (* Halt dosing and activate alarm *)
    Flow_B_SP := 0.0;                 (* Stop Reactant B flow *)
    Ratio_Error := FALSE;             (* Clear ratio error *)
    AlarmActive := TRUE;              (* Activate alarm *)
    RETURN;
END_IF;

(* Validate sensor inputs *)
(* Check for numerical stability and valid ranges *)
Valid_Flow_A := Flow_A_PV >= Flow_A_Min AND Flow_A_PV <= Flow_A_Max AND ABS(Flow_A_PV) <= 1.0E10;
Valid_Flow_B := Flow_B_PV >= Flow_B_Min AND Flow_B_PV <= Flow_B_Max AND ABS(Flow_B_PV) <= 1.0E10;

IF NOT Valid_Flow_A OR NOT Valid_Flow_B THEN
    (* Invalid sensor readings: Stop flow *)
    Flow_B_SP := 0.0;                 (* Stop Reactant B flow *)
    Ratio_Error := TRUE;              (* Flag ratio error *)
    AlarmActive := TRUE;              (* Activate alarm *)
    RETURN;
ELSE
    Ratio_Error := FALSE;             (* Clear ratio error *)
    AlarmActive := FALSE;             (* Clear alarm unless ratio error *)
END_IF;

(* Calculate actual ratio *)
(* Avoid division by zero *)
IF Flow_B_PV > 0.0 THEN
    Actual_Ratio := Flow_A_PV / Flow_B_PV; (* Current A:B ratio *)
ELSE
    Actual_Ratio := 0.0;              (* Set to 0 if Flow_B_PV is zero *)
END_IF;

(* Check ratio deviation *)
Ratio_Deviation := ABS(Actual_Ratio - Ratio_Setpoint);
IF Ratio_Deviation > Tolerance THEN
    Ratio_Error := TRUE;              (* Flag ratio error *)
    AlarmActive := TRUE;              (* Activate alarm *)
ELSE
    Ratio_Error := FALSE;             (* Clear ratio error *)
END_IF;

(* Calculate flow setpoint for Reactant B *)
(* Flow_B_SP = Flow_A_PV / Ratio_Setpoint *)
Flow_B_SP := Flow_A_PV / Ratio_Setpoint;

(* Clamp Flow_B_SP within safe operational range *)
IF Flow_B_SP > Flow_B_SP_Max THEN
    Flow_B_SP := Flow_B_SP_Max;       (* Limit to maximum flow *)
ELSIF Flow_B_SP < Flow_B_SP_Min THEN
    Flow_B_SP := Flow_B_SP_Min;       (* Limit to minimum flow *)
END_IF;

(* Ensure safe state on power-up or PLC stop *)
(* No flow if fault or emergency *)
IF EmergencyStop OR Ratio_Error THEN
    Flow_B_SP := 0.0;                 (* Stop Reactant B flow *)
END_IF;

END_PROGRAM
