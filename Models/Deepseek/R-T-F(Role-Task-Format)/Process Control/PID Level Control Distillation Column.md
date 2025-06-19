PROGRAM LiquidLevelControl
VAR
    // Inputs
    Level_PV : REAL; // Measured level (%)
    Level_SP : REAL := 60.0; // Setpoint (%)

    // PID tuning parameters
    Kp : REAL := 1.5;
    Ki : REAL := 0.4;
    Kd : REAL := 0.2;

    // Internal PID variables
    Error : REAL;
    Prev_Error : REAL := 0.0;
    Integral : REAL := 0.0;
    Derivative : REAL;
    Valve_Position : REAL;

    // Valve output limits
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
    Error := Level_SP - Level_PV;

    // Update integral term
    Integral := Integral + Error * 0.1; // Integral over 100 ms

    // Update derivative term
    Derivative := (Error - Prev_Error) / 0.1; // Derivative over 100 ms

    // Calculate PID output
    Valve_Position := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

    // Clamp output to safe range
    IF Valve_Position > Valve_Max THEN
        Valve_Position := Valve_Max;
    ELSIF Valve_Position < Valve_Min THEN
        Valve_Position := Valve_Min;
    END_IF;

    // Store current error as previous error for next iteration
    Prev_Error := Error;

    // Reset the timer
    SamplingTimer(IN := FALSE);
END_IF;

// Start the timer
SamplingTimer(IN := TRUE);

// Valve_Position is sent to valve control interface
// Example: SetValvePosition(Valve_Position); // Placeholder function to set valve position
