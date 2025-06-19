FUNCTION_BLOCK FB_RatioControl_2to1
VAR_INPUT
    // Measured flows
    Flow_A_PV : REAL; // Reactant A flow rate
    Flow_B_PV : REAL; // Reactant B flow rate

    // Ratio control parameters
    Ratio_Setpoint : REAL := 2.0; // Target A:B ratio
    Tolerance : REAL := 0.05;     // Acceptable deviation from setpoint

    // Optional enable flag
    Enable_Control : BOOL := TRUE;
END_VAR

VAR_OUTPUT
    // Calculated outputs
    Flow_B_SP : REAL;            // Setpoint for Flow B controller
    Actual_Ratio : REAL;         // Current calculated A:B ratio
    Ratio_OK : BOOL;             // TRUE if within tolerance
    Ratio_Error : REAL;          // Deviation from setpoint
END_VAR

VAR
    // Internal variables
    Error : REAL;
END_VAR

// Only execute logic if control is enabled
IF Enable_Control THEN

    // --- STEP 1: Calculate actual A:B ratio safely ---
    IF Flow_B_PV <> 0.0 THEN
        Actual_Ratio := Flow_A_PV / Flow_B_PV;
    ELSE
        Actual_Ratio := 0.0;
    END_IF;

    // --- STEP 2: Compute new setpoint for Flow B ---
    IF Ratio_Setpoint <> 0.0 THEN
        Flow_B_SP := Flow_A_PV / Ratio_Setpoint;
    ELSE
        Flow_B_SP := 0.0; // Avoid division by zero
    END_IF;

    // --- STEP 3: Evaluate ratio error ---
    Ratio_Error := Actual_Ratio - Ratio_Setpoint;

    // --- STEP 4: Check if ratio is within tolerance ---
    Ratio_OK := ABS(Ratio_Error) <= Tolerance;

ELSE
    // If disabled, reset outputs
    Flow_B_SP := 0.0;
    Actual_Ratio := 0.0;
    Ratio_OK := FALSE;
    Ratio_Error := 0.0;
END_IF;

PROGRAM PLC_PRG
VAR
    RatioCtrl : FB_RatioControl_2to1;

    // Inputs
    ReactantA_Flow : REAL := 100.0; // e.g., 100 L/min
    ReactantB_Flow : REAL := 50.0;  // e.g., 50 L/min

    // Outputs
    B_Flow_Setpoint : REAL;
    Current_Ratio : REAL;
    In_Tolerance : BOOL;
    Ratio_Deviation : REAL;
END_VAR

// Call the function block
RatioCtrl(
    Flow_A_PV := ReactantA_Flow,
    Flow_B_PV := ReactantB_Flow,
    Ratio_Setpoint := 2.0,
    Tolerance := 0.05,

    Flow_B_SP => B_Flow_Setpoint,
    Actual_Ratio => Current_Ratio,
    Ratio_OK => In_Tolerance,
    Ratio_Error => Ratio_Deviation
);

// Now you can feed B_Flow_Setpoint into a PID loop or pump speed controller
