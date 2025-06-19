FUNCTION_BLOCK FB_RatioControl
VAR_INPUT
    Flow_A_PV : REAL;         // Measured flow of Reactant A
    Flow_B_PV : REAL;         // Measured flow of Reactant B
    Ratio_Setpoint : REAL := 2.0;  // Desired A:B ratio
    Tolerance : REAL := 0.05; // Allowed deviation
END_VAR

VAR_OUTPUT
    Flow_B_SP : REAL;         // Setpoint for Reactant B flow controller
    Ratio_Alarm : BOOL;       // Alarm flag if ratio exceeds tolerance
END_VAR

VAR
    Actual_Ratio : REAL;
    Error : REAL;
END_VAR

// --- Calculate the actual A:B ratio ---
IF Flow_B_PV > 0.0 THEN
    Actual_Ratio := Flow_A_PV / Flow_B_PV;
ELSE
    Actual_Ratio := 0.0;
END_IF

// --- Compute the required setpoint for Reactant B ---
Flow_B_SP := Flow_A_PV / Ratio_Setpoint;

// --- Evaluate error and set alarm if deviation exceeds tolerance ---
Error := Actual_Ratio - Ratio_Setpoint;

IF ABS(Error) > Tolerance THEN
    Ratio_Alarm := TRUE;
ELSE
    Ratio_Alarm := FALSE;
END_IF
