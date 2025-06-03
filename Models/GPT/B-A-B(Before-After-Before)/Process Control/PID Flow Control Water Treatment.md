VAR
    // === Process Inputs ===
    FlowRate : REAL;           // Water flow in L/min
    Dosing_PV : REAL;          // Measured chlorine concentration in ppm

    // === Setpoint ===
    Dosing_SP : REAL := 3.0;   // Target chlorine level (ppm)

    // === PID Parameters ===
    Kp : REAL := 2.0;
    Ki : REAL := 0.5;
    Kd : REAL := 0.1;

    // === Internal PID State ===
    Error : REAL;
    Prev_Error : REAL := 0.0;
    Integral : REAL := 0.0;
    Derivative : REAL;

    // === Output ===
    Dosing_Output : REAL;

    // === Safety Limits ===
    Min_Dose : REAL := 0.0;
    Max_Dose : REAL := 10.0;

    // === Timing ===
    SampleTime : REAL := 0.1; // 100 ms
END_VAR

// === PID Calculation ===
Error := Dosing_SP - Dosing_PV;
Integral := Integral + Error * SampleTime;
Derivative := (Error - Prev_Error) / SampleTime;

Dosing_Output := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

// === Safety Clamping ===
IF Dosing_Output > Max_Dose THEN
    Dosing_Output := Max_Dose;
ELSIF Dosing_Output < Min_Dose THEN
    Dosing_Output := Min_Dose;
END_IF;

Prev_Error := Error;

// === Dosing_Output now controls the chemical dosing pump ===
