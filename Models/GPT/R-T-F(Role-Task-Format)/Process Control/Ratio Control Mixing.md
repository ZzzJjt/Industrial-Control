VAR
    // Inputs
    Flow_A_PV       : REAL;              // Measured flow rate of Reactant A
    Flow_B_PV       : REAL;              // Measured flow rate of Reactant B

    // Output
    Flow_B_SP       : REAL;              // Calculated setpoint for Reactant B

    // Ratio control parameters
    Ratio_Setpoint  : REAL := 2.0;       // Desired A:B ratio (2:1)
    Actual_Ratio    : REAL;              // Measured A:B ratio
    Error           : REAL;              // Ratio deviation
    Tolerance       : REAL := 0.05;      // Acceptable deviation (±)

    // Alarm or status flag
    Ratio_OutOfRange : BOOL := FALSE;    // Indicates if ratio is outside tolerance
END_VAR

// Calculate actual A:B ratio
IF Flow_B_PV > 0.0 THEN
    Actual_Ratio := Flow_A_PV / Flow_B_PV;
ELSE
    Actual_Ratio := 0.0;
END_IF

// Calculate required setpoint for Reactant B to maintain 2:1 ratio
Flow_B_SP := Flow_A_PV / Ratio_Setpoint;

// Calculate error and check against tolerance
Error := Actual_Ratio - Ratio_Setpoint;

IF ABS(Error) > Tolerance THEN
    Ratio_OutOfRange := TRUE;      // Raise flag or trigger alarm
ELSE
    Ratio_OutOfRange := FALSE;     // Ratio is within acceptable range
END_IF
