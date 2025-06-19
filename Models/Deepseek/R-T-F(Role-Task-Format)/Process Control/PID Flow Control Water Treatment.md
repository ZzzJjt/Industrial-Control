PROGRAM ChlorineDosingControl
VAR
    // Inputs
    FlowRate : REAL; // Flow rate (optional for future use)
    Dosing_PV : REAL; // Measured chlorine concentration (ppm)
    Dosing_SP : REAL := 3.0; // Target setpoint (ppm)

    // PID tuning parameters
    Kp : REAL := 2.0;
    Ki : REAL := 0.5;
    Kd : REAL := 0.1;

    // Internal PID variables
    Error : REAL;
    Prev_Error : REAL := 0.0;
    Integral : REAL := 0.0;
    Derivative : REAL;
    Dosing_Output : REAL;

    // Safety limits
    Max_Dose : REAL := 10.0;
    Min_Dose : REAL := 0.0;

    // Timer for sampling every 100 ms
    SamplingTimer : TON;
END_VAR

// Initialize the timer with a period of 100 ms
SamplingTimer(PT := T#100ms);

// Main control loop
IF SamplingTimer.Q THEN
    // Calculate error
    Error := Dosing_SP - Dosing_PV;

    // Update integral term
    Integral := Integral + Error * 0.1; // Integral over 100 ms

    // Update derivative term
    Derivative := (Error - Prev_Error) / 0.1; // Derivative over 100 ms

    // Calculate PID output
    Dosing_Output := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

    // Clamp output to safe range
    IF Dosing_Output > Max_Dose THEN
        Dosing_Output := Max_Dose;
    ELSIF Dosing_Output < Min_Dose THEN
        Dosing_Output := Min_Dose;
    END_IF;

    // Store current error as previous error for next iteration
    Prev_Error := Error;

    // Reset the timer
    SamplingTimer(IN := FALSE);
END_IF;

// Start the timer
SamplingTimer(IN := TRUE);

// Dosing_Output is sent to dosing pump control interface
// Example: SetDosingPump(Dosing_Output); // Placeholder function to set dosing pump speed
