PROGRAM PLC_PRG
TITLE 'Chemical Mixing Feedforward Control – Reactant A Flow Setpoint'

(*
    Description:
    A feedforward control system for a chemical mixing process involving two reactants.
    
    Features:
    - Calculates required flow rate of Reactant A based on measured Reactant B flow
    - Maintains desired stoichiometric or process ratio using feedforward logic
    - Adjusts dynamically to variations in concentration or temperature (optional)
    - Reduces process variability and improves product consistency
    - Modular design suitable for integration with feedback control

    Safety & Performance:
    - Prevents off-spec product due to delayed response
    - Improves stability under variable feed conditions
    - Easy to extend with sensor inputs, diagnostics, or PID loops
*)

// Constants and Tuning Parameters
CONST
    // Desired mixing ratio: Reactant A / Reactant B
    DESIRED_RATIO : REAL := 2.0;     // e.g., 2 parts A per 1 part B

    // Nominal values for optional compensation
    NOMINAL_CONCENTRATION_B : REAL := 0.8;   // Nominal concentration of Reactant B
    NOMINAL_TEMPERATURE_B   : REAL := 25.0;  // Nominal temperature of Reactant B

    // Compensation scaling factors (empirical or lab-calibrated)
    CONC_COMPENSATION_FACTOR : REAL := 0.1;  // How much A needs to change per unit of concentration deviation
    TEMP_COMPENSATION_FACTOR : REAL := 0.05; // How much A needs to change per °C deviation
END_CONST

VAR
    // Inputs
    Flow_B             : REAL := 0.0;         // Measured flow rate of Reactant B (L/min or kg/s)
    Concentration_B    : REAL := 0.8;         // Measured concentration of Reactant B (fraction or %)
    Temperature_B      : REAL := 25.0;        // Measured temperature of Reactant B (°C)

    // Outputs
    Flow_A_Setpoint    : REAL := 0.0;         // Calculated setpoint for Reactant A flow (L/min or kg/s)

    // Internal variables
    Base_Flow_A        : REAL := 0.0;
    Conc_Deviation     : REAL := 0.0;
    Temp_Deviation     : REAL := 0.0;
    Compensation_Factor: REAL := 1.0;

    // Optional: Enable dynamic compensation
    USE_CONC_COMPENSATION : BOOL := TRUE;
    USE_TEMP_COMPENSATION : BOOL := FALSE;
END_VAR

// === MAIN LOGIC ===

// Base feedforward calculation: A = Ratio * B
Base_Flow_A := DESIRED_RATIO * Flow_B;

// Optional concentration-based compensation
IF USE_CONC_COMPENSATION THEN
    Conc_Deviation := Concentration_B - NOMINAL_CONCENTRATION_B;
    Compensation_Factor := 1.0 + (Conc_Deviation * CONC_COMPENSATION_FACTOR);
ELSE
    Compensation_Factor := 1.0;
END_IF;

// Optional temperature-based compensation
IF USE_TEMP_COMPENSATION THEN
    Temp_Deviation := Temperature_B - NOMINAL_TEMPERATURE_B;
    Compensation_Factor := Compensation_Factor + (Temp_Deviation * TEMP_COMPENSATION_FACTOR);
END_IF;

// Final setpoint calculation with compensation
Flow_A_Setpoint := Base_Flow_A * Compensation_Factor;

// At this point, Flow_A_Setpoint can be sent to a flow controller or VFD
// For example: analog output to valve/pump controlling Reactant A

END_PROGRAM
