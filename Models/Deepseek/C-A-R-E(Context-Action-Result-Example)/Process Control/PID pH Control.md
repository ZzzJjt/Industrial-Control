PROGRAM PIDControlIndustrialChemicalProcess
VAR_INPUT
    pH_PV : REAL; // Measured pH value
END_VAR

VAR_OUTPUT
    Dosing_Output : REAL; // Output to drive the acid/base addition pump (% open)
END_VAR

VAR
    // Setpoint
    pH_SP : REAL := 7.0; // Target setpoint

    // PID parameters
    Kp : REAL := 2.5;
    Ki : REAL := 0.6;
    Kd : REAL := 0.3;

    // PID state variables
    Error : REAL;
    Prev_Error : REAL := 0.0;
    Integral : REAL := 0.0;
    Derivative : REAL;

    // Dosing limits (output to pump actuator)
    Dosing_Min : REAL := 0.0;
    Dosing_Max : REAL := 100.0;

    // Sampling Time (100 ms)
    Sample_Time : TIME := T#100ms;
END_VAR

// PID Control Logic
Error := pH_SP - pH_PV;
Integral := Integral + Error * TD_TO_S(Sample_Time);
Derivative := (Error - Prev_Error) / TD_TO_S(Sample_Time);

Dosing_Output := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

// Clamp dosing output to safe limits
IF Dosing_Output > Dosing_Max THEN
    Dosing_Output := Dosing_Max;
ELSIF Dosing_Output < Dosing_Min THEN
    Dosing_Output := Dosing_Min;
END_IF;

Prev_Error := Error;

// Inline comments explaining the logic:
// The program implements PID feedback control for regulating the pH in an industrial chemical process.
// It continuously monitors the measured pH value (pH_PV) and compares it to a defined setpoint (pH_SP).
// The PID controller calculates the dosing output for the acid/base addition pump using tuned PID gains: Kp, Ki, Kd.
// Clamping logic ensures the dosing output remains within safe limits (Dosing_Min, Dosing_Max) to prevent over-dosing or pH extremes.



