VAR
    (* Inputs *)
    Desired_Ratio      : REAL := 2.0;      (* Target mixing ratio A:B *)
    Flow_B             : REAL;          (* Measured flow rate of Reactant B [L/min] *)
    Concentration_B    : REAL := 0.8;      (* Measured concentration of Reactant B [fraction] *)
    Temperature_B      : REAL := 25.0;     (* Measured temperature of Reactant B [°C] *)

    (* Control parameters *)
    Compensation_Factor : REAL := 1.0;     (* Dynamic compensation factor *)
    
    (* Output *)
    Flow_A_Setpoint    : REAL;             (* Setpoint for Reactant A flow [L/min] *)

    (* Operational limits *)
    Max_Flow_A         : REAL := 100.0;    (* Maximum flow rate for Reactant A [L/min] *)
    Min_Flow_A         : REAL := 0.0;      (* Minimum flow rate for Reactant A [L/min] *)
    
    (* Constants for compensation *)
    Conc_Nominal       : REAL := 0.8;      (* Nominal concentration of Reactant B *)
    Temp_Nominal       : REAL := 25.0;     (* Nominal temperature of Reactant B [°C] *)
    Conc_Sensitivity   : REAL := 0.5;      (* Compensation sensitivity for concentration *)
    Temp_Sensitivity   : REAL := 0.02;     (* Compensation sensitivity for temperature [1/°C] *)
END_VAR

(* Feedforward control for Reactant A flow *)

(* Calculate dynamic compensation factor *)
(* Adjust for concentration: lower concentration increases Flow_A *)
(* Adjust for temperature: higher temperature reduces Flow_A *)
Compensation_Factor := (Conc_Nominal / Concentration_B) * 
                      (1.0 - Temp_Sensitivity * (Temperature_B - Temp_Nominal));

(* Ensure Compensation_Factor stays within reasonable bounds *)
IF Compensation_Factor > 1.5 THEN
    Compensation_Factor := 1.5;
ELSIF Compensation_Factor < 0.5 THEN
    Compensation_Factor := 0.5;
END_IF;

(* Calculate required flow for Reactant A *)
Flow_A_Setpoint := Desired_Ratio * Flow_B * Compensation_Factor;

(* Clamp Flow_A_Setpoint to operational limits *)
IF Flow_A_Setpoint > Max_Flow_A THEN
    Flow_A_Setpoint := Max_Flow_A;
ELSIF Flow_A_Setpoint < Min_Flow_A THEN
    Flow_A_Setpoint := Min_Flow_A;
END_IF;

(* Flow_A_Setpoint is sent to the flow valve or pump controller *)
(* Example: Write Flow_A_Setpoint to analog output for flow control *)
