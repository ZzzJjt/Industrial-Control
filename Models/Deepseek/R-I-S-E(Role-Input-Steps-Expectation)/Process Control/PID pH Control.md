FUNCTION_BLOCK FB_pH_Control_PID
VAR_INPUT
    // Process Variable - Measured pH value
    pH_PV : REAL;

    // Setpoint - Target pH
    pH_SP : REAL := 7.0;

    // PID Tuning Parameters
    Kp : REAL := 2.5;
    Ki : REAL := 0.6;
    Kd : REAL := 0.3;

    // Sampling Time (seconds, e.g., T#100ms = 0.1s)
    Sample_Time : REAL := 0.1;

    // Dosing Output Limits (% open or % dosing rate)
    Dosing_Min : REAL := 0.0;
    Dosing_Max : REAL := 100.0;
END_VAR

VAR_OUTPUT
    // Calculated output to the dosing pump or valve
    Dosing_Output : REAL;
END_VAR

VAR
    // Internal Variables
    Error : REAL;
    Prev_Error : REAL := 0.0;
    Integral : REAL := 0.0;
    Derivative : REAL := 0.0;

    // Anti-Windup Flag
    Integral_Limited : BOOL := FALSE;
END_VAR

// --- STEP 1: Calculate error ---
Error := pH_SP - pH_PV;

// --- STEP 2: Update integral term with anti-windup ---
Integral_Limited := FALSE;

IF (Dosing_Output > Dosing_Max AND Error > 0) OR 
   (Dosing_Output < Dosing_Min AND Error < 0) THEN
    Integral_Limited := TRUE;
ELSE
    Integral := Integral + Error * Sample_Time;
END_IF;

// --- STEP 3: Compute derivative ---
Derivative := (Error - Prev_Error) / Sample_Time;

// --- STEP 4: Apply PID formula ---
Dosing_Output := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

// --- STEP 5: Clamp output to dosing limits ---
IF Dosing_Output > Dosing_Max THEN
    Dosing_Output := Dosing_Max;
ELSIF Dosing_Output < Dosing_Min THEN
    Dosing_Output := Dosing_Min;
END_IF;

// --- STEP 6: Save current error for next cycle ---
Prev_Error := Error;

PROGRAM PLC_PRG
VAR
    pH_Controller : FB_pH_Control_PID;

    // Simulated Inputs
    Current_pH : REAL := 6.9; // Measured pH
    Dosing_Command : REAL;
END_VAR

// Call the function block
pH_Controller(
    pH_PV := Current_pH,
    pH_SP := 7.0,
    Kp := 2.5,
    Ki := 0.6,
    Kd := 0.3,
    Sample_Time := 0.1,

    Dosing_Output => Dosing_Command
);

