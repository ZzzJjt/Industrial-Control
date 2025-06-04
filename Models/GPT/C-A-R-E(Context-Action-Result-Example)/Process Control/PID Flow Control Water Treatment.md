PROGRAM ChlorineDosingPID
VAR
    // --- Input Process Variables ---
    FlowRate        : REAL;             // Water flow rate (L/min)
    Dosing_PV       : REAL;             // Measured chlorine concentration (ppm)
    Dosing_SP       : REAL := 3.0;      // Chlorine concentration setpoint (ppm)

    // --- PID Control Variables ---
    Error           : REAL;
    Prev_Error      : REAL := 0.0;
    Integral        : REAL := 0.0;
    Derivative      : REAL;
    Dosing_Output   : REAL;

    // --- PID Tuning Parameters ---
    Kp              : REAL := 2.0;      // Proportional gain
    Ki              : REAL := 0.5;      // Integral gain
    Kd              : REAL := 0.1;      // Derivative gain

    // --- Sampling Time ---
    Ts              : REAL := 0.1;      // 100 ms in seconds

    // --- Safety Clamps ---
    Max_Dose        : REAL := 10.0;     // Maximum allowable dose
    Min_Dose        : REAL := 0.0;      // Minimum allowable dose
END_VAR

// --- PID Control Computation ---
Error := Dosing_SP - Dosing_PV;

// Accumulate integral term
Integral := Integral + Error * Ts;

// Calculate derivative term
Derivative := (Error - Prev_Error) / Ts;

// PID output
Dosing_Output := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

// Clamp output within safety limits
IF Dosing_Output > Max_Dose THEN
    Dosing_Output := Max_Dose;
ELSIF Dosing_Output < Min_Dose THEN
    Dosing_Output := Min_Dose;
END_IF

// Store error for next cycle
Prev_Error := Error;

// --- Dosing_Output is sent to the dosing pump actuator/controller ---
