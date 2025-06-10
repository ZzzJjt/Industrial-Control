PROGRAM CascadePressureControl_Refinery
VAR
    // --- Outer Loop: Pressure Control ---
    Pressure_SP     : REAL := 12.0;   // Desired pressure setpoint in bar
    Pressure_PV     : REAL;           // Measured vessel pressure
    Pressure_Error  : REAL;           // Error between setpoint and process variable
    Flow_SP         : REAL;           // Output of outer loop: flow setpoint

    // --- Inner Loop: Flow Control ---
    Flow_PV         : REAL;           // Measured inflow rate
    Flow_Error      : REAL;           // Error between flow setpoint and measured flow
    Flow_Output     : REAL;           // Output to control valve or pump

    // --- Controller Gains ---
    Kp_Outer        : REAL := 1.2;    // Gain for outer (pressure) loop
    Kp_Inner        : REAL := 2.5;    // Gain for inner (flow) loop
END_VAR

// --- Outer Loop Execution (Slower Loop) ---
// Generates flow setpoint to correct pressure deviation
Pressure_Error := Pressure_SP - Pressure_PV;
Flow_SP := Kp_Outer * Pressure_Error;

// --- Inner Loop Execution (Faster Loop) ---
// Adjusts valve based on flow setpoint from outer loop
Flow_Error := Flow_SP - Flow_PV;
Flow_Output := Kp_Inner * Flow_Error;

// Flow_Output is the control signal to oil inflow actuator (e.g., analog output to valve or VFD)
