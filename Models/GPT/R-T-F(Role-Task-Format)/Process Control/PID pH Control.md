VAR
    // Inputs
    pH_PV           : REAL;              // Measured pH value
    pH_SP           : REAL := 7.0;       // Desired pH setpoint

    // PID tuning parameters
    Kp              : REAL := 2.5;       // Proportional gain
    Ki              : REAL := 0.6;       // Integral gain
    Kd              : REAL := 0.3;       // Derivative gain

    // Internal PID variables
    Error           : REAL;              // Current error
    Prev_Error      : REAL := 0.0;       // Previous error for derivative calculation
    Integral        : REAL := 0.0;       // Accumulated integral term
    Derivative      : REAL;              // Rate of change of error
    Dosing_Output   : REAL;              // Output signal to dosing system (% rate)

    // Output constraints
    Dosing_Min      : REAL := 0.0;       // Minimum safe dosing output
    Dosing_Max      : REAL := 100.0;     // Maximum safe dosing output

    // Sampling time
    DeltaT          : REAL := 0.1;       // 100 ms sampling interval
END_VAR

// PID control logic (executed every 100 ms)
Error := pH_SP - pH_PV;
Integral := Integral + (Error * DeltaT);
Derivative := (Error - Prev_Error) / DeltaT;

Dosing_Output := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

// Clamp output to maintain safe dosing levels
IF Dosing_Output > Dosing_Max THEN
    Dosing_Output := Dosing_Max;
ELSIF Dosing_Output < Dosing_Min THEN
    Dosing_Output := Dosing_Min;
END_IF

// Update error history for next derivative calculation
Prev_Error := Error;
