FUNCTION_BLOCK FB_ChlorineDosingPID
VAR_INPUT
    // Measured Inputs
    Dosing_PV : REAL;       // Measured chlorine concentration (ppm)
    FlowRate : REAL;        // Current water flow rate (L/min)

    // Setpoint and Tuning
    Dosing_SP : REAL := 3.0; // Chlorine setpoint (ppm)
    Kp : REAL := 2.0;        // Proportional gain
    Ki : REAL := 0.5;        // Integral gain
    Kd : REAL := 0.1;        // Derivative gain

    // Sampling interval (in seconds, e.g., T#100ms = 0.1s)
    Sample_Time : REAL := 0.1;

    // Safety Limits
    Min_Dose : REAL := 0.0;
    Max_Dose : REAL := 10.0;
END_VAR

VAR_OUTPUT
    // Output to dosing pump
    Dosing_Output : REAL;
END_VAR

VAR
    // Internal Control Variables
    Error : REAL;
    Prev_Error : REAL := 0.0;
    Integral : REAL := 0.0;
    Derivative : REAL := 0.0;

    // Optional: Anti-windup flag
    Integral_Limited : BOOL := FALSE;
END_VAR

// --- STEP 1: Calculate Error ---
Error := Dosing_SP - Dosing_PV;

// --- STEP 2: Update Integral Term with Anti-Windup ---
Integral_Limited := FALSE;

IF (Dosing_Output < Min_Dose AND Error > 0) OR (Dosing_Output > Max_Dose AND Error < 0) THEN
    Integral_Limited := TRUE;
ELSE
    Integral := Integral + Error * Sample_Time;
END_IF;

// --- STEP 3: Compute Derivative Term ---
Derivative := (Error - Prev_Error) / Sample_Time;

// --- STEP 4: Apply PID Formula ---
Dosing_Output := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

// --- STEP 5: Clamp Output to Safe Range ---
IF Dosing_Output > Max_Dose THEN
    Dosing_Output := Max_Dose;
ELSIF Dosing_Output < Min_Dose THEN
    Dosing_Output := Min_Dose;
END_IF;

// --- STEP 6: Update Previous Error for Next Cycle ---

PROGRAM PLC_PRG
VAR
    ChlorineCtrl : FB_ChlorineDosingPID;

    // Simulated Inputs
    MeasuredChlorine : REAL := 2.8;
    CurrentFlow : REAL := 50.0;

    // Output
    DosingPumpCommand : REAL;
END_VAR

// Call the function block
ChlorineCtrl(
    Dosing_PV := MeasuredChlorine,
    FlowRate := CurrentFlow,

    Dosing_SP := 3.0,
    Kp := 2.0,
    Ki := 0.5,
    Kd := 0.1,

    Dosing_Output => DosingPumpCommand
);
Prev_Error := Error;
