PROGRAM CascadePressureControl
VAR_INPUT
    Pressure_PV : REAL; // Measured process variable for pressure
    Flow_PV : REAL; // Measured process variable for flow
END_VAR

VAR_OUTPUT
    Flow_Output : REAL; // Output signal to control the valve actuator
END_VAR

VAR
    // Outer loop: pressure control variables
    Pressure_SP : REAL := 12.0; // Desired pressure setpoint
    Pressure_Error : REAL; // Error between pressure setpoint and process variable
    Flow_SP : REAL; // Setpoint for flow based on pressure control

    // Inner loop: flow control variables
    Flow_Error : REAL; // Error between flow setpoint and process variable

    // Proportional gains
    Kp_Outer : REAL := 1.2; // Proportional gain for outer loop
    Kp_Inner : REAL := 2.5; // Proportional gain for inner loop

    // Timing variables
    LastTimeOuter : TIME; // Variable to store the last execution time for outer loop
    LastTimeInner : TIME; // Variable to store the last execution time for inner loop
    CycleTimeOuter : TIME := T#500ms; // Control cycle time for outer loop
    CycleTimeInner : TIME := T#100ms; // Control cycle time for inner loop
END_VAR

// Outer loop: pressure regulation
IF TONR(T:=CycleTimeOuter, ET=>LastTimeOuter).Q THEN
    LastTimeOuter := TONR(T:=CycleTimeOuter, ET=>LastTimeOuter).ET;

    // Calculate the pressure error
    Pressure_Error := Pressure_SP - Pressure_PV;
    
    // Convert the pressure error to a flow setpoint using proportional control
    Flow_SP := Kp_Outer * Pressure_Error;
    
    // Clamp Flow_SP to a reasonable range (e.g., 0 to 100%)
    IF Flow_SP < 0.0 THEN
        Flow_SP := 0.0;
    ELSIF Flow_SP > 100.0 THEN
        Flow_SP := 100.0;
    END_IF;
END_IF;

// Inner loop: flow regulation
IF TONR(T:=CycleTimeInner, ET=>LastTimeInner).Q THEN
    LastTimeInner := TONR(T:=CycleTimeInner, ET=>LastTimeInner).ET;

    // Calculate the flow error
    Flow_Error := Flow_SP - Flow_PV;
    
    // Generate a valve control signal using proportional control
    Flow_Output := Kp_Inner * Flow_Error;
    
    // Clamp Flow_Output to a reasonable range (e.g., 0 to 100%)
    IF Flow_Output < 0.0 THEN
        Flow_Output := 0.0;
    ELSIF Flow_Output > 100.0 THEN
        Flow_Output := 100.0;
    END_IF;
END_IF;

// Additional comments for clarity
// - Outer Loop (Primary Control – Pressure):
//   - Calculate the pressure error as the difference between Pressure_SP and Pressure_PV.
//   - Convert the pressure error to a flow setpoint (Flow_SP) using Kp_Outer.
// - Inner Loop (Secondary Control – Flow):
//   - Calculate the flow error as the difference between Flow_SP and Flow_PV.
//   - Generate a valve control signal (Flow_Output) using Kp_Inner.
// - Real-Time Behavior:
//   - Run the outer loop every 500 ms and the inner loop every 100 ms.
//   - Ensure the inner loop reacts quickly to changes in flow.
// - Clamping Flow_SP and Flow_Output:
//   - Ensure the flow setpoint and valve control signal remain within valid ranges (0 to 100%).




