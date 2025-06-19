PROGRAM LevelControl
VAR
    // --- Process Inputs ---
    Level_PV        : REAL;             // Measured liquid level (%)
    Level_SP        : REAL := 60.0;     // Desired setpoint (%)

    // --- PID Tuning Parameters ---
    Kp              : REAL := 1.5;      // Proportional gain
    Ki              : REAL := 0.4;      // Integral gain
    Kd              : REAL := 0.2;      // Derivative gain

    // --- PID Internal Variables ---
    Error           : REAL;
    Prev_Error      : REAL := 0.0;
    Integral        : REAL := 0.0;
    Derivative      : REAL;
    Valve_Position  : REAL;

    // --- Sampling Time ---
    Ts              : REAL := 0.1;      // Sampling time in seconds (100 ms)

    // --- Output Clamping Limits ---
    Valve_Min       : REAL := 0.0;      // Minimum valve opening (%)
    Valve_Max       : REAL := 100.0;    // Maximum valve opening (%)
END_VAR

// --- PID Control Logic ---
Error := Level_SP - Level_PV;

// Compute Integral and Derivative
Integral := Integral + Error * Ts;
Derivative := (Error - Prev_Error) / Ts;

// Compute PID Output
Valve_Position := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

// Clamp Output Within Safe Limits
IF Valve_Position > Valve_Max THEN
    Valve_Position := Valve_Max;
ELSIF Valve_Position < Valve_Min THEN
    Valve_Position := Valve_Min;
END_IF

// Update for Next Cycle
Prev_Error := Error;

// --- Valve_Position is used to drive inlet control valve ---
