VAR
    // === Inputs ===
    pH_PV : REAL;                 // Measured pH value
    pH_SP : REAL := 7.0;          // Desired pH setpoint (neutral)

    // === PID Tuning Parameters ===
    Kp : REAL := 2.5;             // Proportional gain
    Ki : REAL := 0.6;             // Integral gain
    Kd : REAL := 0.3;             // Derivative gain

    // === PID Internal State ===
    Error : REAL;
    Prev_Error : REAL := 0.0;
    Integral : REAL := 0.0;
    Derivative : REAL;
    Dosing_Output : REAL;         // Output to acid/base pump

    // === Dosing Safety Limits ===
    Dosing_Min : REAL := 0.0;     // 0% pump speed
    Dosing_Max : REAL := 100.0;   // 100% pump speed

    // === Sample Time (in seconds) ===
    SampleTime : REAL := 0.1;     // 100 ms
END_VAR

// === PID Calculation ===
Error := pH_SP - pH_PV;
Integral := Integral + Error * SampleTime;
Derivative := (Error - Prev_Error) / SampleTime;

Dosing_Output := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

// === Output Clamping ===
IF Dosing_Output > Dosing_Max THEN
    Dosing_Output := Dosing_Max;
ELSIF Dosing_Output < Dosing_Min THEN
    Dosing_Output := Dosing_Min;
END_IF;

Prev_Error := Error;
