PROGRAM PressureControl
VAR
    // Inputs
    Pressure_PV : REAL; // Measured reactor pressure (bar)
    Pressure_SP : REAL := 5.0; // Pressure setpoint (bar)

    // PID gains
    Kp : REAL := 2.0;
    Ki : REAL := 0.8;
    Kd : REAL := 0.3;

    // Internal PID variables
    Error : REAL;
    Prev_Error : REAL := 0.0;
    Integral : REAL := 0.0;
    Derivative : REAL;
    Valve_Output : REAL;

    // Safety constraints
    Valve_Min : REAL := 0.0;
    Valve_Max : REAL := 100.0;

    // Timer for sampling every 100 ms
    SamplingTimer : TON;
END_VAR

// Initialize the timer with a period of 100 ms
SamplingTimer(PT := T#100ms);

// Main control loop
IF SamplingTimer.Q THEN
    // Calculate error
    Error := Pressure_SP - Pressure_PV;

    // Update integral term
    Integral := Integral + Error * 0.1; // Integral over 100 ms

    // Update derivative term
    Derivative := (Error - Prev_Error) / 0.1; // Derivative over 100 ms

    // Calculate PID output
    Valve_Output := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

    // Clamp output to safe range
    IF Valve_Output > Valve_Max THEN
        Valve_Output := Valve_Max;
    ELSIF Valve_Output < Valve_Min THEN
        Valve_Output := Valve_Min;
    END_IF;

    // Store current error as previous error for next iteration
    Prev_Error := Error;

    // Reset the timer
    SamplingTimer(IN := FALSE);
END_IF;

// Start the timer
SamplingTimer(IN := TRUE);

// Valve_Output is sent to valve control interface
// Example: SetValvePosition(Valve_Output); // Placeholder function to set valve position
