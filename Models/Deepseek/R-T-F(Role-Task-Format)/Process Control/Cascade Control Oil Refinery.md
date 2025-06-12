PROGRAM PressureRegulationControl
VAR
    // Outer loop (pressure control)
    Pressure_SP : REAL := 12.0; // Desired pressure setpoint (bar)
    Pressure_PV : REAL;         // Measured pressure process variable (bar)
    Pressure_Error : REAL;      // Error between setpoint and process variable
    Flow_SP : REAL;             // Setpoint for flow rate from outer loop

    // Inner loop (flow control)
    Flow_PV : REAL;              // Measured flow rate process variable (L/min)
    Flow_Error : REAL;           // Error between setpoint and process variable
    Flow_Output : REAL;          // Output to control valve or pump (0 to 100%)

    // Proportional gains
    Kp_Outer : REAL := 1.2;     // Proportional gain for outer loop
    Kp_Inner : REAL := 2.5;     // Proportional gain for inner loop

    // Timers for execution rates
    OuterLoopTimer : TON;       // Timer for outer loop (e.g., 1 second)
    InnerLoopTimer : TON;       // Timer for inner loop (e.g., 0.2 seconds)
END_VAR

// Initialize timers with specified periods
OuterLoopTimer(PT := T#1s); // Outer loop runs every 1 second
InnerLoopTimer(PT := T#200ms); // Inner loop runs every 200 milliseconds

// Outer loop logic: pressure control
IF OuterLoopTimer.Q THEN
    Pressure_Error := Pressure_SP - Pressure_PV; // Calculate error for outer loop
    Flow_SP := Kp_Outer * Pressure_Error; // Adjust flow setpoint based on outer loop error
    OuterLoopTimer(IN := FALSE); // Reset outer loop timer
END_IF;

// Inner loop logic: flow control
IF InnerLoopTimer.Q THEN
    Flow_Error := Flow_SP - Flow_PV; // Calculate error for inner loop
    Flow_Output := Kp_Inner * Flow_Error; // Calculate control valve/pump output

    // Ensure Flow_Output is within safe limits (e.g., 0 to 100%)
    IF Flow_Output < 0.0 THEN
        Flow_Output := 0.0;
    ELSIF Flow_Output > 100.0 THEN
        Flow_Output := 100.0;
    END_IF;

    InnerLoopTimer(IN := FALSE); // Reset inner loop timer
END_IF;

// Start timers
OuterLoopTimer(IN := TRUE);
InnerLoopTimer(IN := TRUE);

// Flow_Output drives the oil flow actuator (e.g., valve or pump)
// Example: ControlValveOrPump(Flow_Output); // Placeholder function to actuate the control valve or pump
