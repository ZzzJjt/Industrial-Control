PROGRAM PIDChlorineDosingControl
VAR_INPUT
    Dosing_PV : REAL; // Measured chlorine concentration in ppm
END_VAR

VAR_OUTPUT
    Dosing_Output : REAL; // Output signal to control the dosing pump
END_VAR

VAR
    Dosing_SP : REAL := 3.0; // Setpoint in ppm

    // PID tuning parameters
    Kp : REAL := 2.0;
    Ki : REAL := 0.5;
    Kd : REAL := 0.1;

    // Internal variables
    Error : REAL;
    Prev_Error : REAL := 0.0;
    Integral : REAL := 0.0;
    Derivative : REAL;

    // Safety limits
    Min_Dose : REAL := 0.0;
    Max_Dose : REAL := 10.0;

    // Timing variables
    LastTime : TIME; // Variable to store the last execution time
    CycleTime : TIME := T#100ms; // Control cycle time
END_VAR

// Check if the control cycle time has elapsed
IF TONR(T:=CycleTime, ET=>LastTime).Q THEN
    LastTime := TONR(T:=CycleTime, ET=>LastTime).ET;

    // Calculate the error between setpoint and process variable
    Error := Dosing_SP - Dosing_PV;

    // Accumulate the integral term
    Integral := Integral + Error * T_TO_S(CycleTime);

    // Compute the derivative term
    Derivative := (Error - Prev_Error) / T_TO_S(CycleTime);

    // Calculate the PID output
    Dosing_Output := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

    // Clamp the output within safe operational limits
    IF Dosing_Output > Max_Dose THEN
        Dosing_Output := Max_Dose;
    ELSIF Dosing_Output < Min_Dose THEN
        Dosing_Output := Min_Dose;
    END_IF;

    // Update previous error for the next iteration
    Prev_Error := Error;
END_IF;

// Additional comments for clarity
// - Read the measured chlorine concentration (Dosing_PV).
// - Calculate the error as the difference between the setpoint (Dosing_SP) and the process variable (Dosing_PV).
// - Apply PID logic using predefined gains (Kp, Ki, Kd):
//   - Proportional: Directly proportional to the current error.
//   - Integral: Accumulates past errors over time.
//   - Derivative: Predicts future error by considering the rate of change of the error.
// - Update internal states: accumulate the integral and compute the derivative based on the error change.
// - Clamp the output (Dosing_Output) between Min_Dose and Max_Dose to ensure safe operation.
// - Send the result to control the dosing pump.



