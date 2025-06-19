FUNCTION_BLOCK FB_ChlorineDosingPID
VAR_INPUT
    FlowRate      : REAL;     // Measured flow rate (L/min)
    Dosing_PV     : REAL;     // Measured chlorine concentration (ppm)
    Dosing_SP     : REAL := 3.0; // Setpoint: 3 ppm
END_VAR

VAR_OUTPUT
    Dosing_Output : REAL;     // Output signal to dosing pump
END_VAR

VAR
    // PID internal states
    Error         : REAL;
    Prev_Error    : REAL := 0.0;
    Integral      : REAL := 0.0;
    Derivative    : REAL;
    
    // PID tuning parameters
    Kp            : REAL := 2.0;
    Ki            : REAL := 0.5;
    Kd            : REAL := 0.1;
    
    // Time step
    dt            : REAL := 0.1; // 100 ms sample time
    
    // Output limits
    Max_Dose      : REAL := 10.0;
    Min_Dose      : REAL := 0.0;
END_VAR

// --- PID calculation ---
Error := Dosing_SP - Dosing_PV;
Integral := Integral + Error * dt;
Derivative := (Error - Prev_Error) / dt;

Dosing_Output := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

// --- Clamp dosing output to safe range ---
IF Dosing_Output > Max_Dose THEN
    Dosing_Output := Max_Dose;
ELSIF Dosing_Output < Min_Dose THEN
    Dosing_Output := Min_Dose;
END_IF;

Prev_Error := Error;
