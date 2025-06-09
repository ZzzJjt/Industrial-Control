FUNCTION_BLOCK FB_PID_pHControl
VAR_INPUT
    pH_PV : REAL;            // Measured pH value
    pH_SP : REAL := 7.0;     // Desired pH setpoint
END_VAR

VAR_OUTPUT
    Dosing_Output : REAL;    // Output to acid/base dosing system
END_VAR

VAR
    // PID Parameters
    Kp : REAL := 2.5;        // Proportional gain
    Ki : REAL := 0.6;        // Integral gain
    Kd : REAL := 0.3;        // Derivative gain

    // Control internals
    Error       : REAL;
    Prev_Error  : REAL := 0.0;
    Integral    : REAL := 0.0;
    Derivative  : REAL;

    // Output constraints
    Dosing_Min : REAL := 0.0;
    Dosing_Max : REAL := 100.0;

    // Sample time (100 ms = 0.1 s)
    dt : REAL := 0.1;
END_VAR

// --- PID Control Logic ---
Error := pH_SP - pH_PV;
Integral := Integral + (Error * dt);
Derivative := (Error - Prev_Error) / dt;

Dosing_Output := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

// --- Clamp Dosing Output to Safe Limits ---
IF Dosing_Output > Dosing_Max THEN
    Dosing_Output := Dosing_Max;
ELSIF Dosing_Output < Dosing_Min THEN
    Dosing_Output := Dosing_Min;
END_IF;

Prev_Error := Error;
