VAR
    // === Process Inputs ===
    Level_PV : REAL;               // Current measured liquid level (%)
    Level_SP : REAL := 60.0;       // Desired setpoint level (%)

    // === PID Tuning Parameters ===
    Kp : REAL := 1.5;
    Ki : REAL := 0.4;
    Kd : REAL := 0.2;

    // === PID Internal State ===
    Error : REAL;
    Prev_Error : REAL := 0.0;
    Integral : REAL := 0.0;
    Derivative : REAL;

    // === Output ===
    Valve_Position : REAL;

    // === Output Limits for Valve Actuation ===
    Valve_Min : REAL := 0.0;       // Fully closed
    Valve_Max : REAL := 100.0;     // Fully open

    // === Timing ===
    SampleTime : REAL := 0.1;      // 100 ms
END_VAR

// === PID Calculation ===
Error := Level_SP - Level_PV;
Integral := Integral + Error * SampleTime;
Derivative := (Error - Prev_Error) / SampleTime;

Valve_Position := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

// === Clamp Valve Position to Safe Range ===
IF Valve_Position > Valve_Max THEN
    Valve_Position := Valve_Max;
ELSIF Valve_Position < Valve_Min THEN
    Valve_Position := Valve_Min;
END_IF;

Prev_Error := Error;

// === Valve_Position is the control signal to the inlet valve actuator ===
