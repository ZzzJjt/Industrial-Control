PROGRAM RatioControl_2to1
VAR
    // --- Input Variables ---
    Flow_A_PV       : REAL;              // Measured flow rate of Reactant A (e.g., L/min)
    Flow_B_PV       : REAL;              // Measured flow rate of Reactant B (e.g., L/min)

    // --- Output Variable ---
    Flow_B_SP       : REAL;              // Setpoint for Reactant B flow to maintain 2:1 ratio

    // --- Control Parameters ---
    Ratio_Setpoint  : REAL := 2.0;       // Desired ratio A:B = 2:1
    Tolerance        : REAL := 0.05;     // Acceptable deviation margin

    // --- Monitoring Variables ---
    Actual_Ratio    : REAL;
    Error           : REAL;
    Ratio_Alarm     : BOOL := FALSE;     // TRUE if ratio deviates beyond tolerance
END_VAR

// --- Ratio Calculation ---
IF Flow_B_PV > 0.0 THEN
    Actual_Ratio := Flow_A_PV / Flow_B_PV;
ELSE
    Actual_Ratio := 0.0;  // Avoid divide-by-zero
END_IF

// --- Calculate Desired Flow for Reactant B ---
Flow_B_SP := Flow_A_PV / Ratio_Setpoint;

// --- Check for Deviation ---
Error := Actual_Ratio - Ratio_Setpoint;

// --- Raise Alarm if Deviation is Out of Bounds ---
IF ABS(Error) > Tolerance THEN
    Ratio_Alarm := TRUE;   // Trigger alarm or corrective logic
ELSE
    Ratio_Alarm := FALSE;
END_IF

// Flow_B_SP can now be passed to a PID controller or flow control actuator
