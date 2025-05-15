PROGRAM PIDControlGasTurbineTemperature
VAR_INPUT
    Temp_PV : REAL; // Measured turbine temperature (°C)
END_VAR

VAR_OUTPUT
    Valve_Position : REAL; // Output to drive the inlet valve (% open)
END_VAR

VAR
    // Setpoint
    Temp_SP : REAL := 950.0; // Desired setpoint (°C)

    // PID tuning parameters
    Kp : REAL := 3.0;
    Ki : REAL := 0.7;
    Kd : REAL := 0.2;

    // Internal control variables
    Error : REAL;
    Prev_Error : REAL := 0.0;
    Integral : REAL := 0.0;
    Derivative : REAL;

    // Valve constraints
    Valve_Min : REAL := 0.0;
    Valve_Max : REAL := 100.0;

    // Sampling Time (100 ms)
    Sample_Time : TIME := T#100ms;
END_VAR

// PID Control Logic
Error := Temp_SP - Temp_PV;
Integral := Integral + Error * TD_TO_S(Sample_Time);
Derivative := (Error - Prev_Error) / TD_TO_S(Sample_Time);

Valve_Position := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

// Clamp valve output to safe limits
IF Valve_Position > Valve_Max THEN
    Valve_Position := Valve_Max;
ELSIF Valve_Position < Valve_Min THEN
    Valve_Position := Valve_Min;
END_IF;

Prev_Error := Error;

// Inline comments explaining the logic:
// The program implements PID feedback control for regulating the internal temperature of a gas turbine.
// It uses a temperature sensor input (Temp_PV) and compares it to a setpoint (Temp_SP).
// The PID controller calculates the valve opening for the inlet valve using tuned PID gains: Kp, Ki, Kd.
// Clamping logic ensures the valve position remains within safe limits (Valve_Min, Valve_Max) to prevent overheating or undercooling.



