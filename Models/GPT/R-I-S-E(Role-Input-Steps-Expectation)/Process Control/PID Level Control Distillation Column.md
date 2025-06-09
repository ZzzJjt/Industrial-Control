FUNCTION_BLOCK FB_Level_PID
VAR_INPUT
    Level_PV     : REAL;       // Measured level (%)
    Level_SP     : REAL := 60.0; // Setpoint level (%)
END_VAR

VAR_OUTPUT
    Valve_Position : REAL;     // Output to inlet valve (0–100%)
END_VAR

VAR
    // PID internal states
    Error       : REAL;
    Prev_Error  : REAL := 0.0;
    Integral    : REAL := 0.0;
    Derivative  : REAL;

    // PID tuning parameters
    Kp          : REAL := 1.5;
    Ki          : REAL := 0.4;
    Kd          : REAL := 0.2;

    // Execution interval (100 ms)
    dt          : REAL := 0.1;

    // Valve limits
    Valve_Min   : REAL := 0.0;
    Valve_Max   : REAL := 100.0;
END_VAR

// --- PID Control Logic ---
Error := Level_SP - Level_PV;
Integral := Integral + Error * dt;
Derivative := (Error - Prev_Error) / dt;

Valve_Position := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

// --- Clamp output to valve range ---
IF Valve_Position > Valve_Max THEN
    Valve_Position := Valve_Max;
ELSIF Valve_Position < Valve_Min THEN
    Valve_Position := Valve_Min;
END_IF;

Prev_Error := Error;
