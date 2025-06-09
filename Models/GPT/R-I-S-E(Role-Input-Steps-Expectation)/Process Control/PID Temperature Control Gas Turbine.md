FUNCTION_BLOCK FB_PID_TurbineTempControl
VAR_INPUT
    Temp_PV : REAL;         // Measured turbine inlet temperature (°C)
    Temp_SP : REAL := 950.0; // Temperature setpoint (°C)
END_VAR

VAR_OUTPUT
    Valve_Position : REAL;  // Output to control turbine inlet valve (% open)
END_VAR

VAR
    // PID Tuning Parameters
    Kp : REAL := 3.0;
    Ki : REAL := 0.7;
    Kd : REAL := 0.2;

    // PID Internals
    Error       : REAL;
    Prev_Error  : REAL := 0.0;
    Integral    : REAL := 0.0;
    Derivative  : REAL;

    // Control Valve Limits
    Valve_Min : REAL := 0.0;
    Valve_Max : REAL := 100.0;

    // Sampling Period
    dt : REAL := 0.1; // 100 ms
END_VAR

// --- PID Computation ---
Error := Temp_SP - Temp_PV;
Integral := Integral + Error * dt;
Derivative := (Error - Prev_Error) / dt;

Valve_Position := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

// --- Clamp Output ---
IF Valve_Position > Valve_Max THEN
    Valve_Position := Valve_Max;
ELSIF Valve_Position < Valve_Min THEN
    Valve_Position := Valve_Min;
END_IF;

Prev_Error := Error;
