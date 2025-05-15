PROGRAM PIDControlDistillationColumn
VAR_INPUT
    Level_PV : REAL;       // Current liquid level (%)
END_VAR

VAR_OUTPUT
    Valve_Position : REAL; // Output to drive the inlet valve (% open)
END_VAR

VAR
    // Setpoint
    Level_SP : REAL := 60.0; // Setpoint for liquid level (%)

    // PID tuning parameters
    Kp : REAL := 1.5;
    Ki : REAL := 0.4;
    Kd : REAL := 0.2;

    // PID internals
    Error : REAL;
    Prev_Error : REAL := 0.0;
    Integral : REAL := 0.0;
    Derivative : REAL;

    // Safety limits
    Valve_Min : REAL := 0.0;
    Valve_Max : REAL := 100.0;

    // Sampling Time (100 ms)
    Sample_Time : TIME := T#100ms;
END_VAR

// PID Control Logic
Error := Level_SP - Level_PV;
Integral := Integral + Error * TD_TO_S(Sample_Time);
Derivative := (Error - Prev_Error) / TD_TO_S(Sample_Time);

Valve_Position := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

// Clamp valve position to safe limits
IF Valve_Position > Valve_Max THEN
    Valve_Position := Valve_Max;
ELSIF Valve_Position < Valve_Min THEN
    Valve_Position := Valve_Min;
END_IF;

Prev_Error := Error;

// Inline comments explaining the logic:
// The program implements PID feedback control for regulating the liquid level in a distillation column.
// It uses a level transmitter input (Level_PV) and compares it to a setpoint (Level_SP).
// The PID controller calculates the valve opening for the inlet stream using tuned PID gains: Kp, Ki, Kd.
// Clamping logic ensures the valve position remains within safe limits (Valve_Min, Valve_Max) to prevent overfeeding or underfeeding.



