VAR
    // Process values
    Pressure_PV       : REAL;           // Measured reactor pressure (bar)
    Pressure_SP       : REAL := 5.0;    // Desired pressure setpoint (bar)

    // PID tuning parameters
    Kp                : REAL := 2.0;    // Proportional gain
    Ki                : REAL := 0.8;    // Integral gain
    Kd                : REAL := 0.3;    // Derivative gain

    // PID internal variables
    Error             : REAL;
    Prev_Error        : REAL := 0.0;
    Integral          : REAL := 0.0;
    Derivative        : REAL;
    Valve_Output      : REAL;           // Output signal to valve actuator

    // Output limits for valve position
    Valve_Min         : REAL := 0.0;    // Fully closed
    Valve_Max         : REAL := 100.0;  // Fully open

    // Control cycle interval (100 ms)
    SampleTime        : REAL := 0.1;
END_VAR

// -----------------------------
// PID CONTROL EXECUTION
// -----------------------------

// Compute current error
Error := Pressure_SP - Pressure_PV;

// Update integral
Integral := Integral + (Error * SampleTime);

// Calculate derivative
Derivative := (Error - Prev_Error) / SampleTime;

// PID output calculation
Valve_Output := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

// Clamp valve output to safety range
IF Valve_Output > Valve_Max THEN
    Valve_Output := Valve_Max;
ELSIF Valve_Output < Valve_Min THEN
    Valve_Output := Valve_Min;
END_IF

// Save current error for next loop
Prev_Error := Error;
