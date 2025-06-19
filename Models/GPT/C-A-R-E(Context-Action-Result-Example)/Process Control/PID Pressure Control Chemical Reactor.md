PROGRAM ReactorPressureControl
VAR
    // --- Inputs ---
    Pressure_PV     : REAL;             // Measured reactor pressure (bar)
    Pressure_SP     : REAL := 5.0;      // Desired pressure setpoint (bar)

    // --- PID Tuning Parameters ---
    Kp              : REAL := 2.0;      // Proportional gain
    Ki              : REAL := 0.8;      // Integral gain
    Kd              : REAL := 0.3;      // Derivative gain

    // --- PID Internal Variables ---
    Error           : REAL;
    Prev_Error      : REAL := 0.0;
    Integral        : REAL := 0.0;
    Derivative      : REAL;
    Valve_Output    : REAL;             // Final output to valve

    // --- Sampling Time ---
    Ts              : REAL := 0.1;      // 100 ms sampling time

    // --- Output Limits ---
    Valve_Min       : REAL := 0.0;      // Fully closed
    Valve_Max       : REAL := 100.0;    // Fully open
END_VAR

// --- PID Control Logic ---
Error := Pressure_SP - Pressure_PV;

Integral := Integral + Error * Ts;
Derivative := (Error - Prev_Error) / Ts;

Valve_Output := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

// --- Clamp the output for safety ---
IF Valve_Output > Valve_Max THEN
    Valve_Output := Valve_Max;
ELSIF Valve_Output < Valve_Min THEN
    Valve_Output := Valve_Min;
END_IF

Prev_Error := Error;

// --- Valve_Output is sent to pressure control actuator (e.g., valve) ---
