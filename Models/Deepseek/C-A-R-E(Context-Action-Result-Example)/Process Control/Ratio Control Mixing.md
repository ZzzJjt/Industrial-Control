PROGRAM RatioControlChemicalBlending
VAR_INPUT
    Flow_A_PV : REAL; // Measured flow of reactant A
    Flow_B_PV : REAL; // Measured flow of reactant B
END_VAR

VAR_OUTPUT
    Flow_B_SP : REAL; // Desired setpoint flow for reactant B
    Alarm : BOOL;     // Alarm flag if ratio deviation exceeds tolerance
END_VAR

VAR
    // Setpoint ratio A:B = 2:1
    Ratio_Setpoint : REAL := 2.0;

    // Actual ratio calculated from process values
    Actual_Ratio : REAL;

    // Error calculation for ratio deviation
    Error : REAL;

    // Tolerance for acceptable ratio deviation
    Tolerance : REAL := 0.05;
END_VAR

// Calculate actual ratio if Flow_B_PV > 0
IF Flow_B_PV > 0.0 THEN
    Actual_Ratio := Flow_A_PV / Flow_B_PV;
ELSE
    Actual_Ratio := 0.0;
END_IF;

// Determine desired flow for reactant B
Flow_B_SP := Flow_A_PV / Ratio_Setpoint;

// Optional error check
Error := Actual_Ratio - Ratio_Setpoint;

// Raise alarm if absolute error exceeds tolerance
Alarm := ABS(Error) > Tolerance;

// Inline comments explaining the logic:
// The program implements ratio control for maintaining a fixed 2:1 ratio between reactant A and reactant B.
// It monitors the actual flow rates of reactant A (Flow_A_PV) and reactant B (Flow_B_PV).
// The program calculates the required setpoint for reactant B (Flow_B_SP) to maintain the specified ratio.
// It dynamically updates the Flow_B_SP value based on changes in Flow_A_PV.
// An optional error check raises an alarm if the actual ratio deviates from the setpoint by more than the specified tolerance.



