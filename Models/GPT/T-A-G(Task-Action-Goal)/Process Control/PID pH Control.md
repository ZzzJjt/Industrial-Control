VAR
    // pH Measurement and Setpoint
    pH_PV           : REAL;            // Measured pH value
    pH_SP           : REAL := 7.0;     // Desired pH setpoint

    // PID Tuning Parameters
    Kp              : REAL := 2.5;     // Proportional gain
    Ki              : REAL := 0.6;     // Integral gain
    Kd              : REAL := 0.3;     // Derivative gain

    // PID Control Variables
    Error           : REAL;
    Prev_Error      : REAL := 0.0;
    Integral        : REAL := 0.0;
    Derivative      : REAL;
    Dosing_Output   : REAL;            // Output signal to dosing actuator

    // Output Limits
    Dosing_Min      : REAL := 0.0;
    Dosing_Max      : REAL := 100.0;

    // Sampling interval (100 ms)
    SampleTime      : REAL := 0.1;
END_VAR

// ----------------------------
// PID CONTROL LOGIC (100 ms)
// ----------------------------

// Step 1: Calculate Error
Error := pH_SP - pH_PV;

// Step 2: Update Integral
Integral := Integral + (Error * SampleTime);

// Step 3: Calculate Derivative
Derivative := (Error - Prev_Error) / SampleTime;

// Step 4: Compute PID Output
Dosing_Output := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

// Step 5: Clamp Output for Safety
IF Dosing_Output > Dosing_Max THEN
    Dosing_Output := Dosing_Max;
ELSIF Dosing_Output < Dosing_Min THEN
    Dosing_Output := Dosing_Min;
END_IF

// Step 6: Store Previous Error
Prev_Error := Error;
