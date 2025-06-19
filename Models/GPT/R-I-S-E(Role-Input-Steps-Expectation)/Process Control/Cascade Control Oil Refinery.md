FUNCTION_BLOCK CascadePressureControl
VAR_INPUT
    Pressure_SP     : REAL := 12.0; // Desired vessel pressure (bar)
    Pressure_PV     : REAL;         // Measured vessel pressure
    Flow_PV         : REAL;         // Measured oil flow rate
END_VAR

VAR_OUTPUT
    Flow_Output     : REAL;         // Command to flow control valve
    Flow_SP         : REAL;         // Inner loop setpoint
END_VAR

VAR
    // Control errors
    Pressure_Error  : REAL;
    Flow_Error      : REAL;

    // Controller gains
    Kp_Outer        : REAL := 1.2;  // Outer loop gain for pressure
    Kp_Inner        : REAL := 2.5;  // Inner loop gain for flow
END_VAR

// --- Outer Loop: Pressure Control ---
Pressure_Error := Pressure_SP - Pressure_PV;
Flow_SP := Kp_Outer * Pressure_Error; // Setpoint for flow controller

// --- Inner Loop: Flow Control ---
Flow_Error := Flow_SP - Flow_PV;
Flow_Output := Kp_Inner * Flow_Error; // Valve actuation signal
