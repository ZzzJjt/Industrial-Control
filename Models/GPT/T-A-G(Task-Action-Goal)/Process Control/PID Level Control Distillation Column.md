VAR
    // Inputs
    Level_PV       : REAL;           // Measured liquid level in %
    Level_SP       : REAL := 60.0;   // Desired setpoint level in %

    // PID tuning parameters
    Kp             : REAL := 1.5;    // Proportional gain
    Ki             : REAL := 0.4;    // Integral gain
    Kd             : REAL := 0.2;    // Derivative gain

    // Internal PID state variables
    Error          : REAL;
    Prev_Error     : REAL := 0.0;
    Integral       : REAL := 0.0;
    Derivative     : REAL;
    Valve_Position : REAL;           // Output signal to valve actuator

    // Valve position constraints
    Valve_Min      : REAL := 0.0;    // Fully closed
    Valve_Max      : REAL := 100.0;  // Fully open

    // Time step (assuming 100 ms control loop)
    SampleTime     : REAL := 0.1;
END_VAR

// -----------------------------
// PID CONTROL CALCULATION
// -----------------------------

// Compute control error
Error := Level_SP - Level_PV;

// Update integral with anti-windup logic (optional enhancement)
Integral := Integral + (Error * SampleTime);

// Calculate derivative
Derivative := (Error - Prev_Error) / SampleTime;

// Compute PID output
Valve_Position := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

// Clamp output to safe valve limits
IF Valve_Position > Valve_Max THEN
    Valve_Position := Valve_Max;
ELSIF Valve_Position < Valve_Min THEN
    Valve_Position := Valve_Min;
END_IF

// Store error for next scan
Prev_Error := Error;
