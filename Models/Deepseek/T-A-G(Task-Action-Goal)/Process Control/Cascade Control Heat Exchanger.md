PROGRAM CascadeTemperatureControl
VAR_INPUT
    Temp_PV : REAL; // Measured process variable for temperature
    Flow_PV : REAL; // Measured process variable for flow
END_VAR

VAR_OUTPUT
    Flow_Output : REAL; // Output signal to control the valve actuator
END_VAR

VAR
    // Outer temperature loop variables
    Temp_SP : REAL := 85.0; // Desired temperature setpoint
    Temp_Error : REAL; // Error between temperature setpoint and process variable
    Flow_SP : REAL; // Setpoint for flow based on temperature control

    // Inner flow loop variables
    Flow_Error : REAL; // Error between flow setpoint and process variable

    // Proportional gains
    Kp_Outer : REAL := 1.0; // Proportional gain for outer loop
    Kp_Inner : REAL := 2.0; // Proportional gain for inner loop

    // Timing variables
    LastTime : TIME; // Variable to store the last execution time
    CycleTime : TIME := T#100ms; // Control cycle time
END_VAR

// Check if the control cycle time has elapsed
IF TONR(T:=CycleTime, ET=>LastTime).Q THEN
    LastTime := TONR(T:=CycleTime, ET=>LastTime).ET;

    // Outer loop: calculate flow setpoint from temperature control
    Temp_Error := Temp_SP - Temp_PV;
    Flow_SP := Kp_Outer * Temp_Error;

    // Inner loop: control flow to match flow setpoint
    Flow_Error := Flow_SP - Flow_PV;
    Flow_Output := Kp_Inner * Flow_Error;

    // Clamp Flow_Output to a reasonable range (e.g., 0 to 100%)
    IF Flow_Output < 0.0 THEN
        Flow_Output := 0.0;
    ELSIF Flow_Output > 100.0 THEN
        Flow_Output := 100.0;
    END_IF;
END_IF;

// Additional comments for clarity
// - Outer Loop (Temperature Control):
//   - Calculate the temperature error as the difference between Temp_SP and Temp_PV.
//   - Convert the temperature error to a flow setpoint (Flow_SP) using Kp_Outer.
// - Inner Loop (Flow Control):
//   - Calculate the flow error as the difference between Flow_SP and Flow_PV.
//   - Generate a valve control signal (Flow_Output) using Kp_Inner.
// - Real-Time Behavior:
//   - Run both loops cyclically with a 100 ms cycle time.
//   - Ensure the inner loop reacts quickly to changes in flow.
// - Clamping Flow_Output:
//   - Ensure the valve control signal remains within a valid range (0 to 100%).




