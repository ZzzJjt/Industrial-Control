FUNCTION_BLOCK FB_ChemicalMixingFeedforward
VAR_INPUT
    // Mixing Parameters
    Desired_Ratio : REAL := 2.0; // Ratio A:B (e.g., 2:1)

    // Measured Inputs from Reactant B
    Flow_B : REAL;               // Measured flow rate of Reactant B (L/min)
    Concentration_B : REAL := 1.0; // Optional: measured concentration (normalized)
    Temperature_B : REAL := 25.0;  // Optional: measured temperature (°C)

    // Compensation Settings
    Enable_Concentration_Compensation : BOOL := FALSE;
    Enable_Temperature_Compensation : BOOL := FALSE;

    // Tuning factors for optional compensation logic
    Conc_Comp_Factor : REAL := 0.05; // % adjustment per unit deviation in concentration
    Temp_Comp_Reference : REAL := 25.0; // Reference temperature
    Temp_Comp_Factor : REAL := 0.01; // % adjustment per degree deviation
END_VAR

VAR_OUTPUT
    // Calculated setpoint for Reactant A
    Flow_A_Setpoint : REAL;
END_VAR

VAR
    // Internal Variables
    Comp_Factor : REAL := 1.0;
    Conc_Deviation : REAL;
    Temp_Deviation : REAL;
END_VAR

// --- STEP 1: Optional Dynamic Compensation ---
Comp_Factor := 1.0;

// Concentration compensation
IF Enable_Concentration_Compensation THEN
    Conc_Deviation := 1.0 - Concentration_B; // Assuming normalized value (1.0 = nominal)
    Comp_Factor := Comp_Factor * (1.0 + Conc_Comp_Factor * Conc_Deviation);
END_IF;

// Temperature compensation
IF Enable_Temperature_Compensation THEN
    Temp_Deviation := Temperature_B - Temp_Comp_Reference;
    Comp_Factor := Comp_Factor * (1.0 + Temp_Comp_Factor * Temp_Deviation);
END_IF;

// --- STEP 2: Feedforward Calculation ---
Flow_A_Setpoint := Desired_Ratio * Flow_B * Comp_Factor;
