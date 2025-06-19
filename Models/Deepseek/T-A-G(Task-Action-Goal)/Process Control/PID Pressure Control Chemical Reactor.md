PROGRAM PIDPressureControl
VAR_INPUT
    Pressure_PV : REAL; // Measured reactor pressure in bar
END_VAR

VAR_OUTPUT
    Valve_Output : REAL; // Output signal to control the pressure valve in percentage (%)
END_VAR

VAR
    Pressure_SP : REAL := 5.0; // Setpoint pressure in bar

    // PID tuning parameters
    Kp : REAL := 2.0;
    Ki : REAL := 0.8;
    Kd : REAL := 0.3;

    // Internal variables
    Error : REAL;
    Prev_Error : REAL := 0.0;
    Integral : REAL := 0.0;
    Derivative : REAL;

    // Valve output limits
    Valve_Min : REAL := 0.0;
    Valve_Max : REAL := 100.0;

    // Timing variables
    LastTime : TIME; // Variable to store the last execution time
    CycleTime : TIME := T#100ms; // Control cycle time
END_VAR

// Check if the control cycle time has elapsed
IF TONR(T:=CycleTime, ET=>LastTime).Q THEN
    LastTime := TONR(T:=CycleTime, ET=>LastTime).ET;

    // Calculate the error between setpoint and process variable
    Error := Pressure_SP - Pressure_PV;

    // Accumulate the integral term
    Integral := Integral + Error * T_TO_S(CycleTime);

    // Compute the derivative term
    Derivative := (Error - Prev_Error) / T_TO_S(CycleTime);

    // Calculate the PID output
    Valve_Output := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

    // Clamp the output within safe operational limits
    IF Valve_Output > Valve_Max THEN
        Valve_Output := Valve_Max;
    ELSIF Valve_Output < Valve_Min THEN
        Valve_Output := Valve_Min;
    END_IF;

    // Update previous error for the next iteration
    Prev_Error := Error;
END_IF;

// Additional comments for clarity
// - Read the measured reactor pressure (Pressure_PV).
// - Calculate the error as the difference between the setpoint (Pressure_SP) and the process variable (Pressure_PV).
// - Apply PID logic using predefined gains (Kp, Ki, Kd):
//   - Proportional: Directly proportional to the current error.
//   - Integral: Accumulates past errors over time.
//   - Derivative: Predicts future error by considering the rate of change of the error.
// - Update internal states: accumulate the integral and compute the derivative based on the error change.
// - Clamp the output (Valve_Output) between Valve_Min and Valve_Max to ensure safe operation.
// - Send the result to control the pressure valve.



