FUNCTION_BLOCK FB_PID_PressureControl
VAR_INPUT
    Pressure_PV : REAL;         // Measured pressure (bar)
    Pressure_SP : REAL := 5.0;  // Desired setpoint (bar)
END_VAR

VAR_OUTPUT
    Valve_Output : REAL;        // Valve position output (0–100%)
END_VAR

VAR
    // PID tuning parameters
    Kp : REAL := 2.0;
    Ki : REAL := 0.8;
    Kd : REAL := 0.3;

    // PID internal variables
    Error      : REAL;
    Prev_Error : REAL := 0.0;
    Integral   : REAL := 0.0;
    Derivative : REAL;

    // Valve limits
    Valve_Min : REAL := 0.0;
    Valve_Max : REAL := 100.0;

    // Time step (sampling interval)
    dt : REAL := 0.1; // 100 ms
END_VAR

// --- PID Logic ---
Error := Pressure_SP - Pressure_PV;
Integral := Integral + Error * dt;
Derivative := (Error - Prev_Error) / dt;

Valve_Output := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

// --- Clamp valve output ---
IF Valve_Output > Valve_Max THEN
    Valve_Output := Valve_Max;
ELSIF Valve_Output < Valve_Min THEN
    Valve_Output := Valve_Min;
END_IF;

Prev_Error := Error;
