VAR
    // Process Variables
    Temp_PV         : REAL;            // Measured turbine temperature (°C)
    Temp_SP         : REAL := 950.0;   // Desired setpoint (°C)

    // PID Gains
    Kp              : REAL := 3.0;     // Proportional gain
    Ki              : REAL := 0.7;     // Integral gain
    Kd              : REAL := 0.2;     // Derivative gain

    // PID Internals
    Error           : REAL;
    Prev_Error      : REAL := 0.0;
    Integral        : REAL := 0.0;
    Derivative      : REAL;
    Valve_Position  : REAL;            // Output to inlet valve actuator

    // Output Constraints
    Valve_Min       : REAL := 0.0;     // 0% valve opening
    Valve_Max       : REAL := 100.0;   // 100% valve opening

    // Sampling interval (100 ms)
    SampleTime      : REAL := 0.1;
END_VAR

// ----------------------------
// PID CONTROL LOGIC
// ----------------------------

// Calculate error
Error := Temp_SP - Temp_PV;

// Update integral term
Integral := Integral + (Error * SampleTime);

// Calculate derivative term
Derivative := (Error - Prev_Error) / SampleTime;

// Compute PID output
Valve_Position := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

// Clamp valve output to safe range
IF Valve_Position > Valve_Max THEN
    Valve_Position := Valve_Max;
ELSIF Valve_Position < Valve_Min THEN
    Valve_Position := Valve_Min;
END_IF

// Store current error for next cycle
Prev_Error := Error;
