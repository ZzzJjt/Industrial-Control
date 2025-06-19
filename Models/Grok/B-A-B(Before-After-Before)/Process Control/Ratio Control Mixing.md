(* Chemical Mixing Ratio Control in IEC 61131-3 Structured Text *)
(* Purpose: Maintain 2:1 ratio of reactants A:B by dynamically adjusting flow rates *)

PROGRAM ChemicalRatioControl
VAR
    (* Inputs *)
    Flow_A_PV : REAL;               (* Measured flow rate of reactant A in L/min *)
    Flow_B_PV : REAL;               (* Measured flow rate of reactant B in L/min *)

    (* Outputs *)
    Flow_B_SP : REAL;               (* Setpoint for reactant B flow rate in L/min *)

    (* Control Parameters *)
    Ratio_Setpoint : REAL := 2.0;   (* Desired ratio A:B = 2:1 *)
    Tolerance : REAL := 0.05;       (* Acceptable ratio error margin *)
    Max_Flow_B : REAL := 50.0;      (* Maximum allowable flow for B in L/min *)
    Min_Flow_B : REAL := 0.0;       (* Minimum allowable flow for B in L/min *)
    Max_Flow_A : REAL := 100.0;     (* Maximum expected flow for A in L/min *)

    (* Internal Variables *)
    Actual_Ratio : REAL;            (* Current measured ratio A:B *)
    Error : REAL;                   (* Deviation from target ratio *)
    Ratio_Fault : BOOL;             (* TRUE if ratio error exceeds tolerance *)
    Input_Fault : BOOL;             (* TRUE if flow inputs are invalid *)
END_VAR

(* Main Control Logic *)
(* 1. Input Validation *)
IF Flow_A_PV < 0.0 OR Flow_A_PV > Max_Flow_A OR Flow_B_PV < 0.0 OR Flow_B_PV > Max_Flow_B THEN
    Input_Fault := TRUE;
    Flow_B_SP := Min_Flow_B;    (* Set safe default during fault *)
ELSE
    Input_Fault := FALSE;

    (* 2. Calculate Actual Ratio *)
    IF Flow_B_PV > 0.0 THEN
        Actual_Ratio := Flow_A_PV / Flow_B_PV;  (* Compute current A:B ratio *)
    ELSE
        Actual_Ratio := 0.0;    (* Avoid division by zero *)
    END_IF;

    (* 3. Calculate Target Flow for Reactant B *)
    Flow_B_SP := Flow_A_PV / Ratio_Setpoint;  (* Set B flow to maintain 2:1 ratio *)

    (* 4. Clamp Flow Setpoint to Safe Limits *)
    IF Flow_B_SP > Max_Flow_B THEN
        Flow_B_SP := Max_Flow_B;    (* Prevent over-flow *)
    ELSIF Flow_B_SP < Min_Flow_B THEN
        Flow_B_SP := Min_Flow_B;    (* Prevent negative flow *)
    END_IF;

    (* 5. Error Detection *)
    Error := Actual_Ratio - Ratio_Setpoint;  (* Calculate ratio deviation *)
    IF ABS(Error) > Tolerance THEN
        Ratio_Fault := TRUE;    (* Flag significant ratio deviation *)
    ELSE
        Ratio_Fault := FALSE;
    END_IF;
END_IF;

(* Notes:
   - Ratio Control:
     - Maintains 2:1 ratio (A:B) by setting Flow_B_SP based on Flow_A_PV
     - Flow_B_SP dynamically adjusts to match Flow_A_PV / Ratio_Setpoint
   - Safety:
     - Input_Fault triggers for invalid flows (negative or exceeding Max_Flow_A/B)
     - Flow_B_SP clamped between Min_Flow_B (0.0) and Max_Flow_B (50.0 L/min)
     - Ratio_Fault flags deviations exceeding Tolerance (0.05)
   - Physical Integration:
     - Flow_A_PV, Flow_B_PV: Analog inputs from flow sensors
     - Flow_B_SP: Analog output to pump or valve controller for B (e.g., 4–20 mA)
   - Scalability:
     - Adjust Ratio_Setpoint for different mixing ratios
     - Integrate with PID loop for finer Flow_B control
   - Maintenance:
     - Add HMI to display Flow_A_PV, Flow_B_PV, Flow_B_SP, Actual_Ratio, Ratio_Fault
     - Log Ratio_Fault and Input_Fault for diagnostics
*)
END_PROGRAM
