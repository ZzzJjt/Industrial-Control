PROGRAM PHControl
VAR
    // Inputs
    pH_PV : REAL; // Measured pH value
    pH_SP : REAL := 7.0; // Target pH

    // PID gains
    Kp : REAL := 2.5;
    Ki : REAL := 0.6;
    Kd : REAL := 0.3;

    // Internal PID variables
    Error : REAL;
    Prev_Error : REAL := 0.0;
    Integral : REAL := 0.0;
    Derivative : REAL;
    Dosing_Output : REAL;

    // Output limits
    Dosing_Min : REAL := 0.0;
    Dosing_Max : REAL := 100.0;

    // Timer for sampling every 100 ms
    SamplingTimer : TON;
END_VAR

// Initialize the timer with a period of 100 ms
SamplingTimer(PT := T#100ms);

// Main control loop
IF SamplingTimer.Q THEN
    // Calculate error
    Error := pH_SP - pH_PV;

    // Update integral term
    Integral := Integral + Error * 0.1; // Integral over 100 ms

    // Update derivative term
    Derivative := (Error - Prev_Error) / 0.1; // Derivative over 100 ms

    // Calculate PID output
    Dosing_Output := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

    // Clamp output to safe range
    IF Dosing_Output > Dosing_Max THEN
        Dosing_Output := Dosing_Max;
    ELSIF Dosing_Output < Dosing_Min THEN
        Dosing_Output := Dosing_Min;
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
