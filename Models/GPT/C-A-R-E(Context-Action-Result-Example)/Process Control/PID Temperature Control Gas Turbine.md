PROGRAM GasTurbineTempControl
VAR
    // --- Inputs ---
    Temp_PV         : REAL;              // Measured turbine temperature (°C)
    Temp_SP         : REAL := 950.0;     // Target temperature setpoint (°C)

    // --- PID Tuning Parameters ---
    Kp              : REAL := 3.0;       // Proportional gain
    Ki              : REAL := 0.7;       // Integral gain
    Kd              : REAL := 0.2;       // Derivative gain

    // --- PID Internal State ---
    Error           : REAL;
    Prev_Error      : REAL := 0.0;
    Integral        : REAL := 0.0;
    Derivative      : REAL;
    Valve_Position  : REAL;              // Control output to inlet valve

    // --- Sampling Time ---
    Ts              : REAL := 0.1;       // 100 ms

    // --- Valve Output Limits ---
    Valve_Min       : REAL := 0.0;       // Fully closed
    Valve_Max       : REAL := 100.0;     // Fully open
END_VAR

// --- PID Control Execution ---
Error := Temp_SP - Temp_PV;

Integral := Integral + Error * Ts;
Derivative := (Error - Prev_Error) / Ts;

Valve_Position := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

// --- Enforce Output Bounds ---
IF Valve_Position > Valve_Max THEN
    Valve_Position := Valve_Max;
ELSIF Valve_Position < Valve_Min THEN
    Valve_Position := Valve_Min;
END_IF

Prev_Error := Error;

// --- Valve_Position is used to modulate the gas inlet valve actuator ---
