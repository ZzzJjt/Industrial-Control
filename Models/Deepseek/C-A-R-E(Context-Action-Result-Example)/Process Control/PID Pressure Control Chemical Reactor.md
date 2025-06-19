PROGRAM PIDControlChemicalReactorPressure
VAR_INPUT
    Pressure_PV : REAL; // Measured pressure (bar)
END_VAR

VAR_OUTPUT
    Valve_Output : REAL; // Output to drive the pressure control valve (% open)
END_VAR

VAR
    // Setpoint
    Pressure_SP : REAL := 5.0; // Setpoint pressure (bar)

    // PID tuning parameters
    Kp : REAL := 2.0;
    Ki : REAL := 0.8;
    Kd : REAL := 0.3;

    // Internal control variables
    Error : REAL;
    Prev_Error : REAL := 0.0;
    Integral : REAL := 0.0;
    Derivative : REAL;

    // Output constraints
    Valve_Min : REAL := 0.0;
    Valve_Max : REAL := 100.0;

    // Sampling Time (100 ms)
    Sample_Time : TIME := T#100ms;
END_VAR

// PID Control Logic
Error := Pressure_SP - Pressure_PV;
Integral := Integral + Error * TD_TO_S(Sample_Time);
Derivative := (Error - Prev_Error) / TD_TO_S(Sample_Time);

Valve_Output := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

// Clamp valve output to safe limits
IF Valve_Output > Valve_Max THEN
    Valve_Output := Valve_Max;
ELSIF Valve_Output < Valve_Min THEN
    Valve_Output := Valve_Min;
END_IF;

Prev_Error := Error;

// Inline comments explaining the logic:
// The program implements PID feedback control for regulating the internal pressure in a chemical reactor.
// It uses a pressure transmitter input (Pressure_PV) and compares it to a setpoint (Pressure_SP).
// The PID controller calculates the valve opening for the pressure control valve using tuned PID gains: Kp, Ki, Kd.
// Clamping logic ensures the valve position remains within safe limits (Valve_Min, Valve_Max) to prevent over-pressurization or under-pressurization.



