PROGRAM RatioControl
VAR
    // Inputs
    Flow_A_PV : REAL; // Measured flow rate of Reactant A
    Flow_B_PV : REAL; // Measured flow rate of Reactant B

    // Outputs
    Flow_B_SP : REAL; // Setpoint for Reactant B (output)

    // Constants
    Ratio_Setpoint : REAL := 2.0;     // Desired ratio A:B = 2:1
    Tolerance : REAL := 0.05;         // Allowable deviation

    // Internal variables
    Actual_Ratio : REAL;
    Error : REAL;
    Deviation_Flag : BOOL;            // Flag to indicate deviation from setpoint
END_VAR

// Calculate actual ratio (A:B)
IF Flow_B_PV > 0.0 THEN
    Actual_Ratio := Flow_A_PV / Flow_B_PV;
ELSE
    Actual_Ratio := 0.0;
END_IF;

// Compute required flow setpoint for Reactant B
Flow_B_SP := Flow_A_PV / Ratio_Setpoint;

// Optional: Calculate ratio error for monitoring
Error := Actual_Ratio - Ratio_Setpoint;

// Check if the error exceeds the allowable tolerance
IF ABS(Error) > Tolerance THEN
    Deviation_Flag := TRUE;           // Set flag indicating deviation
ELSE
    Deviation_Flag := FALSE;          // Clear flag
END_IF;

// Deviation_Flag can be used to trigger alerts or corrective actions
// Example: IF Deviation_Flag THEN AlertDeviation(); END_IF; // Placeholder function to handle deviation
