PROGRAM CascadeControlSystem
VAR
    // Outer loop: temperature control
    Temp_SP : REAL := 85.0; // Desired temperature setpoint (°C)
    Temp_PV : REAL;         // Process variable for temperature (measured value)
    Temp_Error : REAL;      // Error between setpoint and process variable
    Flow_SP : REAL;          // Setpoint for flow rate from outer loop

    // Inner loop: flow control
    Flow_PV : REAL;          // Process variable for flow rate (measured value)
    Flow_Error : REAL;       // Error between setpoint and process variable
    Flow_Output : REAL;      // Output to control valve

    // Controller tuning parameters
    Kp_Outer : REAL := 1.0; // Proportional gain for outer loop
    Kp_Inner : REAL := 2.0; // Proportional gain for inner loop
END_VAR

// Outer loop logic
Temp_Error := Temp_SP - Temp_PV; // Calculate error for outer loop
Flow_SP := Kp_Outer * Temp_Error; // Adjust flow setpoint based on outer loop error

// Inner loop logic
Flow_Error := Flow_SP - Flow_PV; // Calculate error for inner loop
Flow_Output := Kp_Inner * Flow_Error; // Calculate control valve output

// Ensure Flow_Output is within safe limits (e.g., 0 to 100%)
IF Flow_Output < 0.0 THEN
    Flow_Output := 0.0;
ELSIF Flow_Output > 100.0 THEN
    Flow_Output := 100.0;
END_IF;

// Flow_Output is used to drive the control valve
// Example: ControlValve(Flow_Output); // Placeholder function to actuate the control valve
