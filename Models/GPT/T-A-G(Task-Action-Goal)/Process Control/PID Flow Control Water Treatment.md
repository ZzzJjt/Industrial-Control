VAR
    // Inputs
    FlowRate        : REAL;            // Optional: flow rate of water (not used in PID directly)
    Dosing_PV       : REAL;            // Measured chlorine concentration [ppm]
    Dosing_SP       : REAL := 3.0;     // Setpoint concentration [ppm]

    // PID tuning parameters
    Kp              : REAL := 2.0;     // Proportional gain
    Ki              : REAL := 0.5;     // Integral gain
    Kd              : REAL := 0.1;     // Derivative gain

    // Internal PID state
    Error           : REAL;
    Prev_Error      : REAL := 0.0;
    Integral        : REAL := 0.0;
    Derivative      : REAL;
    Dosing_Output   : REAL;

    // Safety limits
    Min_Dose        : REAL := 0.0;     // Minimum dosing rate
    Max_Dose        : REAL := 10.0;    // Maximum dosing rate

    // Constants
    SampleTime      : REAL := 0.1;     // 100 ms control loop
END_VAR

// -----------------------------
// PID Control Calculations
// -----------------------------

// Compute error
Error := Dosing_SP - Dosing_PV;

// Integral term with accumulation
Integral := Integral + (Error * SampleTime);

// Derivative term using error difference
Derivative := (Error - Prev_Error) / SampleTime;

// PID output calculation
Dosing_Output := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

// Clamp the dosing output to safety limits
IF Dosing_Output > Max_Dose THEN
    Dosing_Output := Max_Dose;
ELSIF Dosing_Output < Min_Dose THEN
    Dosing_Output := Min_Dose;
END_IF;

// Update previous error for next cycle
Prev_Error := Error;

// Dosing_Output is now ready to be sent to the pump actuator
