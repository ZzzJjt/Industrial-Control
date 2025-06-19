PROGRAM pH_PID_Control
VAR
    // --- Input Variables ---
    pH_PV          : REAL;              // Measured pH value
    pH_SP          : REAL := 7.0;       // Target setpoint (neutral)

    // --- PID Tuning Parameters ---
    Kp             : REAL := 2.5;       // Proportional gain
    Ki             : REAL := 0.6;       // Integral gain
    Kd             : REAL := 0.3;       // Derivative gain

    // --- PID State Variables ---
    Error          : REAL;
    Prev_Error     : REAL := 0.0;
    Integral       : REAL := 0.0;
    Derivative     : REAL;
    Dosing_Output  : REAL;              // Output to acid/base dosing system

    // --- Sampling Time ---
    Ts             : REAL := 0.1;       // 100 ms

    // --- Output Constraints ---
    Dosing_Min     : REAL := 0.0;       // Minimum pump output
    Dosing_Max     : REAL := 100.0;     // Maximum pump output
END_VAR

// --- PID Control Logic (Executed Every 100 ms) ---
Error := pH_SP - pH_PV;
Integral := Integral + Error * Ts;
Derivative := (Error - Prev_Error) / Ts;

Dosing_Output := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

// --- Output Clamping to Prevent Overdosing ---
IF Dosing_Output > Dosing_Max THEN
    Dosing_Output := Dosing_Max;
ELSIF Dosing_Output < Dosing_Min THEN
    Dosing_Output := Dosing_Min;
END_IF

Prev_Error := Error;

// --- Dosing_Output is used to control acid/base addition ---
