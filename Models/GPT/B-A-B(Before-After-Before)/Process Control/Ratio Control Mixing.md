VAR
    // === Process Variables ===
    Flow_A_PV : REAL;              // Measured flow of Reactant A (e.g., L/min)
    Flow_B_PV : REAL;              // Measured flow of Reactant B

    // === Ratio Settings ===
    Ratio_Setpoint : REAL := 2.0;  // Desired A:B ratio
    Tolerance : REAL := 0.05;      // Acceptable error margin

    // === Control Output ===
    Flow_B_SP : REAL;              // Desired flow setpoint for Reactant B

    // === Monitoring ===
    Actual_Ratio : REAL;
    Error : REAL;
    Alarm : BOOL := FALSE;
END_VAR

// === Ratio Calculation ===
IF Flow_B_PV > 0.0 THEN
    Actual_Ratio := Flow_A_PV / Flow_B_PV;
ELSE
    Actual_Ratio := 0.0; // Avoid divide-by-zero
END_IF;

// === Compute Desired B Flow ===
Flow_B_SP := Flow_A_PV / Ratio_Setpoint;

// === Check for Deviation from Ratio ===
Error := Actual_Ratio - Ratio_Setpoint;

IF ABS(Error) > Tolerance THEN
    Alarm := TRUE; // Optional alarm if deviation too large
ELSE
    Alarm := FALSE;
END_IF;

// === Output ===
// Flow_B_SP can now be used as the setpoint for a PID controller or analog output to control pump B
