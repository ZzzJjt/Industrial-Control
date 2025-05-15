PROGRAM PIDControlChlorineDosing
VAR_INPUT
    FlowRate : REAL;       // Current flow rate in L/min
    Dosing_PV : REAL;      // Measured chlorine concentration (ppm)
END_VAR

VAR_OUTPUT
    Dosing_Output : REAL; // Output to drive the dosing pump (e.g., % open)
END_VAR

VAR
    // PID Variables
    Error : REAL;
    Prev_Error : REAL := 0.0;
    Integral : REAL := 0.0;
    Derivative : REAL;

    // Setpoint
    Dosing_SP : REAL := 3.0; // Setpoint for dosing (ppm)

    // Tuning Parameters
    Kp : REAL := 2.0;
    Ki : REAL := 0.5;
    Kd : REAL := 0.1;

    // Safety Limits
    Max_Dose : REAL := 10.0;
    Min_Dose : REAL := 0.0;

    // Sampling Time (100 ms)
    Sample_Time : TIME := T#100ms;
END_VAR

// PID Control Logic
Error := Dosing_SP - Dosing_PV;
Integral := Integral + Error * TD_TO_S(Sample_Time);
Derivative := (Error - Prev_Error) / TD_TO_S(Sample_Time);

Dosing_Output := Kp * Error + Ki * Integral + Kd * Derivative;

// Clamp dosing output to safe limits
IF Dosing_Output > Max_Dose THEN
    Dosing_Output := Max_Dose;
ELSIF Dosing_Output < Min_Dose THEN
    Dosing_Output := Min_Dose;
END_IF;

Prev_Error := Error;

// Inline comments explaining the logic:
// The program implements PID feedback control for regulating chlorine dosing in water treatment systems.
// It maintains a chlorine concentration setpoint of 3 ppm using proportional, integral, and derivative components.
// The sampling time is set to 100 ms for real-time adjustments.
// Safety limits are applied to ensure the dosing rate remains within acceptable boundaries to avoid overdosing or underdosing.



