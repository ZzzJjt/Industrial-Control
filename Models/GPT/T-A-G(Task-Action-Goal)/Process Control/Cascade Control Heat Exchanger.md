VAR
    // Outer Loop (Temperature Control)
    Temp_SP      : REAL := 85.0;     // Temperature setpoint (°C)
    Temp_PV      : REAL;             // Measured temperature (°C)
    Temp_Error   : REAL;             // Error between setpoint and PV
    Flow_SP      : REAL;             // Calculated flow setpoint (L/min)

    // Inner Loop (Flow Control)
    Flow_PV      : REAL;             // Measured flow (L/min)
    Flow_Error   : REAL;             // Error between flow setpoint and PV
    Flow_Output  : REAL;             // Final output signal to control valve (0–100%)

    // Controller Gains
    Kp_Outer     : REAL := 1.5;      // Gain for temperature loop
    Kp_Inner     : REAL := 2.0;      // Gain for flow loop

    // Output Constraints
    MinOutput    : REAL := 0.0;
    MaxOutput    : REAL := 100.0;
END_VAR

// -----------------------------
// Outer Loop: Temperature Control
// -----------------------------
Temp_Error := Temp_SP - Temp_PV;           // Compute temperature error
Flow_SP := Kp_Outer * Temp_Error;          // Convert to flow setpoint

// Optional: Limit Flow_SP for safety
IF Flow_SP < 0.0 THEN
    Flow_SP := 0.0;
ELSIF Flow_SP > 100.0 THEN
    Flow_SP := 100.0;
END_IF

// -----------------------------
// Inner Loop: Flow Control
// -----------------------------
Flow_Error := Flow_SP - Flow_PV;           // Compute flow error
Flow_Output := Kp_Inner * Flow_Error;      // Control signal to flow valve

// Output Clamping
IF Flow_Output < MinOutput THEN
    Flow_Output := MinOutput;
ELSIF Flow_Output > MaxOutput THEN
    Flow_Output := MaxOutput;
END_IF

// Now Flow_Output can be sent to the analog valve actuator
