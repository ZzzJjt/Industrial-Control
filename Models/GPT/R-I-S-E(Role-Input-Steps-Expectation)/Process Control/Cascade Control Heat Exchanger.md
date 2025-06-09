FUNCTION_BLOCK FB_CascadeHeatExchanger
VAR_INPUT
    Temp_SP      : REAL := 85.0;   // Desired outlet temperature [°C]
    Temp_PV      : REAL;           // Measured outlet temperature [°C]
    Flow_PV      : REAL;           // Measured flow rate [e.g., L/min]
END_VAR

VAR_OUTPUT
    Flow_Output  : REAL;           // Output signal to the flow control valve
    Flow_SP      : REAL;           // Flow setpoint from outer loop
END_VAR

VAR
    // Errors
    Temp_Error   : REAL;
    Flow_Error   : REAL;

    // Control gains
    Kp_Outer     : REAL := 1.0;    // Gain for temperature controller (outer loop)
    Kp_Inner     : REAL := 2.0;    // Gain for flow controller (inner loop)
END_VAR

// 1. Outer Loop: Temperature Control
// Compute temperature error and determine desired flow setpoint
Temp_Error := Temp_SP - Temp_PV;
Flow_SP := Kp_Outer * Temp_Error; // Feedforward signal to inner loop

// 2. Inner Loop: Flow Control
// Compute flow error and determine valve output
Flow_Error := Flow_SP - Flow_PV;
Flow_Output := Kp_Inner * Flow_Error; // Final output to flow valve actuator
