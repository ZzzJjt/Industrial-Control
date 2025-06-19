FUNCTION_BLOCK FeedforwardMixerControl
VAR_INPUT
    Flow_B           : REAL;  // Real-time flow rate of Reactant B
    Desired_Ratio    : REAL := 2.0;  // Target A:B ratio, default 2:1
    Concentration_B  : REAL := 0.8;  // Optional compensation input
    Temperature_B    : REAL := 25.0; // Optional compensation input
END_VAR

VAR_OUTPUT
    Flow_A_Setpoint  : REAL;  // Calculated flow rate for Reactant A
END_VAR

VAR
    Compensation_Factor : REAL := 1.0;
END_VAR

// --- Optional dynamic compensation logic (can be customized) ---
// Example: reduce A dosage if B is more concentrated or hotter
IF Concentration_B > 1.0 THEN
    Compensation_Factor := 0.95; // Less A needed
ELSIF Temperature_B > 40.0 THEN
    Compensation_Factor := 0.90; // Lower A for hotter input
ELSE
    Compensation_Factor := 1.0;
END_IF;

// --- Feedforward calculation ---
Flow_A_Setpoint := Desired_Ratio * Flow_B * Compensation_Factor;
