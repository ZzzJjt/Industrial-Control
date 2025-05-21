(* Program: Feedforward Control for Reactant Mixing *)
(* Version: 1.0, Date: 2025-05-21 *)
(* Adjusts Reactant A flow based on Reactant B flow, concentration, and temperature *)
(* Executes cyclically (e.g., 100 ms) with safety checks *)
PROGRAM PRG_FeedforwardMixingControl
VAR
    (* Inputs *)
    Flow_B : REAL;                    (* Measured flow rate of Reactant B, L/min *)
    Concentration_B : REAL;           (* Concentration of Reactant B, mol/L *)
    Temperature_B : REAL;             (* Temperature of Reactant B, °C *)
    EmergencyStop : BOOL;             (* TRUE if emergency stop activated *)
    
    (* Outputs *)
    Flow_A_Setpoint : REAL;           (* Setpoint for Reactant A flow, L/min *)
    InputError : BOOL;                (* TRUE if any input is invalid *)
    AlarmActive : BOOL;               (* TRUE if fault or emergency stop *)
    
    (* Internal Constants *)
    Desired_Ratio : REAL := 2.0;      (* Target A:B flow ratio *)
    Base_Concentration : REAL := 0.8; (* Reference concentration, mol/L *)
    Base_Temperature : REAL := 25.0;  (* Reference temperature, °C *)
    Flow_B_Min : REAL := 0.0;         (* Min valid flow rate, L/min *)
    Flow_B_Max : REAL := 100.0;       (* Max valid flow rate, L/min *)
    Concentration_B_Min : REAL := 0.1; (* Min valid concentration, mol/L *)
    Concentration_B_Max : REAL := 2.0; (* Max valid concentration, mol/L *)
    Temperature_B_Min : REAL := 0.0;   (* Min valid temperature, °C *)
    Temperature_B_Max : REAL := 100.0; (* Max valid temperature, °C *)
    Flow_A_Min : REAL := 0.0;         (* Min valid Flow_A_Setpoint, L/min *)
    Flow_A_Max : REAL := 200.0;       (* Max valid Flow_A_Setpoint, L/min *)
    
    (* Internal Variables *)
    Compensation_Factor : REAL;        (* Compensation for concentration and temperature *)
    ValidFlow_B : BOOL;               (* TRUE if Flow_B is valid *)
    ValidConcentration_B : BOOL;      (* TRUE if Concentration_B is valid *)
    ValidTemperature_B : BOOL;        (* TRUE if Temperature_B is valid *)
END_VAR

(* Initialize outputs *)
Flow_A_Setpoint := 0.0;               (* No flow initially *)
InputError := FALSE;                  (* No initial input error *)
AlarmActive := FALSE;                 (* No initial alarm *)

(* Main logic *)
(* Emergency stop handling: Overrides all operations *)
IF EmergencyStop THEN
    (* Halt system and activate alarm *)
    Flow_A_Setpoint := 0.0;           (* Stop Reactant A flow *)
    InputError := FALSE;              (* Clear input error *)
    AlarmActive := TRUE;              (* Activate alarm *)
    RETURN;
END_IF;

(* Validate sensor inputs *)
(* Check for numerical stability and valid ranges *)
ValidFlow_B := Flow_B >= Flow_B_Min AND Flow_B <= Flow_B_Max AND ABS(Flow_B) <= 1.0E10;
ValidConcentration_B := Concentration_B >= Concentration_B_Min AND Concentration_B <= Concentration_B_Max AND ABS(Concentration_B) <= 1.0E10;
ValidTemperature_B := Temperature_B >= Temperature_B_Min AND Temperature_B <= Temperature_B_Max AND ABS(Temperature_B) <= 1.0E10;

IF NOT ValidFlow_B OR NOT ValidConcentration_B OR NOT ValidTemperature_B THEN
    (* Invalid sensor readings: Stop system *)
    Flow_A_Setpoint := 0.0;           (* Stop Reactant A flow *)
    InputError := TRUE;               (* Flag input error *)
    AlarmActive := TRUE;              (* Activate alarm *)
    RETURN;
ELSE
    InputError := FALSE;              (* Clear input error *)
    AlarmActive := FALSE;             (* Clear alarm *)
END_IF;

(* Calculate compensation factor *)
(* Adjusts for deviations in concentration and temperature *)
(* Example: Concentration compensation scales inversely, temperature linear *)
Compensation_Factor := (Base_Concentration / Concentration_B) * (1.0 + 0.01 * (Temperature_B - Base_Temperature));

(* Ensure compensation factor is within reasonable bounds *)
IF Compensation_Factor < 0.5 THEN
    Compensation_Factor := 0.5;       (* Minimum compensation *)
ELSIF Compensation_Factor > 2.0 THEN
    Compensation_Factor := 2.0;       (* Maximum compensation *)
END_IF;

(* Feedforward control: Calculate Reactant A flow setpoint *)
(* Formula: Flow_A_Setpoint = Desired_Ratio * Flow_B * Compensation_Factor *)
Flow_A_Setpoint := Desired_Ratio * Flow_B * Compensation_Factor;

(* Clamp Flow_A_Setpoint within safe operational range *)
IF Flow_A_Setpoint > Flow_A_Max THEN
    Flow_A_Setpoint := Flow_A_Max;    (* Limit to maximum flow *)
ELSIF Flow_A_Setpoint < Flow_A_Min THEN
    Flow_A_Setpoint := Flow_A_Min;    (* Limit to minimum flow *)
END_IF;

(* Ensure safe state on power-up or PLC stop *)
(* No flow if fault or emergency *)
IF EmergencyStop OR InputError THEN
    Flow_A_Setpoint := 0.0;           (* Stop Reactant A flow *)
END_IF;

END_PROGRAM
