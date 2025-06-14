VAR
    // Measured flow rates
    Flow_A_PV       : REAL;          // Flow of Reactant A (e.g., L/min)
    Flow_B_PV       : REAL;          // Flow of Reactant B

    // Ratio logic
    Flow_B_SP       : REAL;          // Setpoint for Reactant B flow
    Actual_Ratio    : REAL;          // A:B ratio
    Ratio_Setpoint  : REAL := 2.0;   // Desired A:B flow ratio (2:1)
    Error           : REAL;
    Tolerance       : REAL := 0.05;  // Acceptable deviation (5%)

    // Alarm flag
    Ratio_Alarm     : BOOL := FALSE;
END_VAR

// ----------------------------
// RATIO CONTROL LOGIC
// ----------------------------

// Step 1: Calculate actual ratio (A/B)
IF Flow_B_PV > 0.0 THEN
    Actual_Ratio := Flow_A_PV / Flow_B_PV;
ELSE
    Actual_Ratio := 0.0;
END_IF

// Step 2: Calculate desired setpoint for Flow_B
Flow_B_SP := Flow_A_PV / Ratio_Setpoint;

// Step 3: Compare actual vs. desired ratio
Error := Actual_Ratio - Ratio_Setpoint;

// Step 4: Trigger alarm if outside tolerance
IF ABS(Error) > Tolerance THEN
    Ratio_Alarm := TRUE;
ELSE
    Ratio_Alarm := FALSE;
END_IF
