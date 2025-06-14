PROGRAM PIDpHControl
VAR_INPUT
    pH_PV : REAL; // Measured pH value from the sensor
END_VAR

VAR_OUTPUT
    Dosing_Output : REAL; // Output signal to control acid/base dosing in percentage (%)
END_VAR

VAR
    pH_SP : REAL := 7.0; // Desired pH setpoint

    // PID tuning parameters
    Kp : REAL := 2.5;
    Ki : REAL := 0.6;
    Kd : REAL := 0.3;

    // Internal variables
    Error : REAL;
    Prev_Error : REAL := 0.0;
    Integral : REAL := 0.0;
    Derivative : REAL;

    // Dosing output limits
    Dosing_Min : REAL := 0.0;
    Dosing_Max : REAL := 100.0;

    // Timing variables
    LastTime : TIME; // Variable to store the last execution time
    CycleTime : TIME := T#100ms; // Control cycle time
END_VAR

// Check if the control cycle time has elapsed
IF TONR(T:=CycleTime, ET=>LastTime).Q THEN
    LastTime := TONR(T:=CycleTime, ET=>LastTime).ET;

    // Calculate the error between setpoint and process variable
    Error := pH_SP - pH_PV;

    // Accumulate the integral term
    Integral := Integral + Error * T_TO_S(CycleTime);

    // Compute the derivative term
    Derivative := (Error - Prev_Error) / T_TO_S(CycleTime);

    // Calculate the PID output
    Dosing_Output := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

    // Clamp the output within safe operational limits
    IF Dosing_Output > Dosing_Max THEN
        Dosing_Output := Dosing_Max;
    ELSIF Dosing_Output < Dosing_Min THEN
        Dosing_Output := Dosing_Min;
    END_IF;

    // Update previous error for the next iteration
    Prev_Error := Error;
END_IF;

// Additional comments for clarity
// - Read the measured pH value (pH_PV).
// - Calculate the error as the difference between the setpoint (pH_SP) and the process variable (pH_PV).
// - Apply PID logic using predefined gains (Kp, Ki, Kd):
//   - Proportional: Directly proportional to the current error.
//   - Integral: Accumulates past errors over time.
//   - Derivative: Predicts future error by considering the rate of change of the error.
// - Update internal states: accumulate the integral and compute the derivative based on the error change.
// - Clamp the output (Dosing_Output) between Dosing_Min and Dosing_Max to ensure safe operation.
// - Send the result to control the acid/base dosing.



