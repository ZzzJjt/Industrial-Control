VAR
    // Outer Loop – Pressure Control
    Pressure_SP      : REAL := 12.0;       // Pressure Setpoint [bar]
    Pressure_PV      : REAL;               // Measured Pressure [bar]
    Pressure_Error   : REAL;               // Pressure error
    Flow_SP          : REAL;               // Flow Setpoint [m³/h]

    // Inner Loop – Flow Control
    Flow_PV          : REAL;               // Measured Flow [m³/h]
    Flow_Error       : REAL;               // Flow error
    Flow_Output      : REAL;               // Output to flow control valve [%]

    // Tuning Gains
    Kp_Outer         : REAL := 1.2;        // Pressure controller gain
    Kp_Inner         : REAL := 2.5;        // Flow controller gain

    // Safety limits
    MaxFlowOutput    : REAL := 100.0;      // Max valve opening [%]
    MinFlowOutput    : REAL := 0.0;        // Min valve opening [%]
END_VAR

// -----------------------------
// Outer Loop: Pressure Controller
// -----------------------------
Pressure_Error := Pressure_SP - Pressure_PV;     // Calculate pressure error
Flow_SP := Kp_Outer * Pressure_Error;            // Generate flow setpoint

// Optional: Clamp Flow_SP to prevent unrealistic commands
IF Flow_SP < 0.0 THEN
    Flow_SP := 0.0;
ELSIF Flow_SP > 100.0 THEN
    Flow_SP := 100.0;
END_IF

// -----------------------------
// Inner Loop: Flow Controller
// -----------------------------
Flow_Error := Flow_SP - Flow_PV;                 // Calculate flow error
Flow_Output := Kp_Inner * Flow_Error;            // Generate valve output

// Clamp Flow_Output to physical actuator range
IF Flow_Output < MinFlowOutput THEN
    Flow_Output := MinFlowOutput;
ELSIF Flow_Output > MaxFlowOutput THEN
    Flow_Output := MaxFlowOutput;
END_IF

// Flow_Output is the command to the control valve or VFD
