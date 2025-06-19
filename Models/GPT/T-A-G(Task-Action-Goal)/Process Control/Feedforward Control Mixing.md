VAR
    // Inputs
    Desired_Ratio     : REAL := 2.0;    // Target ratio of Reactant A to Reactant B (e.g., 2:1)
    Flow_B            : REAL;           // Measured flow rate of Reactant B [L/min]
    Concentration_B   : REAL := 0.80;   // Optional: Concentration of Reactant B [fraction]
    Temperature_B     : REAL := 25.0;   // Optional: Temperature of Reactant B [°C]

    // Intermediate calculation
    Compensation_Factor : REAL := 1.0;  // Dynamic adjustment factor
    Flow_A_Setpoint     : REAL;         // Final calculated flow rate setpoint for Reactant A [L/min]
END_VAR

// -------------------------
// Optional Compensation Logic
// Adjust factor based on B's concentration and temperature
// -------------------------

IF Concentration_B < 0.75 THEN
    Compensation_Factor := 1.1;  // Compensate for dilution
ELSIF Concentration_B > 0.9 THEN
    Compensation_Factor := 0.95; // Stronger concentration needs less A
ELSE
    Compensation_Factor := 1.0;
END_IF;

// Optional temperature effect (if applicable)
IF Temperature_B < 20.0 THEN
    Compensation_Factor := Compensation_Factor * 1.05;
ELSIF Temperature_B > 35.0 THEN
    Compensation_Factor := Compensation_Factor * 0.95;
END_IF;

// -------------------------
// Feedforward Control Logic
// -------------------------

Flow_A_Setpoint := Desired_Ratio * Flow_B * Compensation_Factor;

// Flow_A_Setpoint should now be sent to actuator for Reactant A (e.g., VFD or control valve)
