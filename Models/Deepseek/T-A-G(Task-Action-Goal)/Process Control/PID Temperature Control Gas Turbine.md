PROGRAM PIDTemperatureControl
VAR_INPUT
    Temp_PV : REAL; // Current turbine temperature in °C
END_VAR

VAR_OUTPUT
    Valve_Position : REAL; // Output signal to control the inlet valve position in percentage (%)
END_VAR

VAR
    Temp_SP : REAL := 950.0; // Temperature setpoint in °C

    // PID tuning parameters
    Kp : REAL := 3.0;
    Ki : REAL := 0.7;
    Kd : REAL := 0.2;

    // Internal variables
    Error : REAL;
    Prev_Error : REAL := 0.0;
    Integral : REAL := 0.0;
    Derivative : REAL;

    // Valve position limits
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
    Error := Temp_SP - Temp_PV;

    // Accumulate the integral term
    Integral := Integral + Error * T_TO_S(CycleTime);

    // Compute the derivative term
    Derivative := (Error - Prev_Error) / T_TO_S(CycleTime);

    // Calculate the PID output
    Valve_Position := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

    // Clamp the output within safe operational limits
    IF Valve_Position > Valve_Max THEN
        Valve_Position := Valve_Max;
    ELSIF Valve_Position < Valve_Min THEN
        Valve_Position := Valve_Min;
    END_IF;

    // Update previous error for the next iteration
    Prev_Error := Error;
END_IF;

// Additional comments for clarity
// - Read the measured turbine temperature (Temp_PV).
// - Calculate the error as the difference between the setpoint (Temp_SP) and the process variable (Temp_PV).
// - Apply PID logic using predefined gains (Kp, Ki, Kd):
//   - Proportional: Directly proportional to the current error.
//   - Integral: Accumulates past errors over time.
//   - Derivative: Predicts future error by considering the rate of change of the error.
// - Update internal states: accumulate the integral and compute the derivative based on the error change.
// - Clamp the output (Valve_Position) between Valve_Min and Valve_Max to ensure safe operation.
// - Send the result to control the inlet valve position.



