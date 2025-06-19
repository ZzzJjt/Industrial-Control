VAR
    // Inputs
    FlowRate        : REAL;              // Flow rate of water (optional for future use)
    Dosing_PV       : REAL;              // Measured chlorine concentration (ppm)
    Dosing_SP       : REAL := 3.0;       // Desired setpoint for chlorine concentration (ppm)

    // PID tuning parameters
    Kp              : REAL := 2.0;       // Proportional gain
    Ki              : REAL := 0.5;       // Integral gain
    Kd              : REAL := 0.1;       // Derivative gain

    // Internal PID variables
    Error           : REAL;              // Current error
    Prev_Error      : REAL := 0.0;       // Previous error (for derivative)
    Integral        : REAL := 0.0;       // Accumulated integral
    Derivative      : REAL;              // Derivative term
    Dosing_Output   : REAL;              // Output command for dosing (ppm)

    // Safety limits
    Max_Dose        : REAL := 10.0;      // Maximum allowable dose
    Min_Dose        : REAL := 0.0;       // Minimum allowable dose

    // Time step (sampling interval)
    DeltaT          : REAL := 0.1;       // 100 ms cycle time
END_VAR

// PID algorithm (executed every 100 ms)
Error := Dosing_SP - Dosing_PV;
Integral := Integral + (Error * DeltaT);
Derivative := (Error - Prev_Error) / DeltaT;

Dosing_Output := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

// Clamp output to safety limits
IF Dosing_Output > Max_Dose THEN
    Dosing_Output := Max_Dose;
ELSIF Dosing_Output < Min_Dose THEN
    Dosing_Output := Min_Dose;
END_IF

// Store current error for next cycle
Prev_Error := Error;
