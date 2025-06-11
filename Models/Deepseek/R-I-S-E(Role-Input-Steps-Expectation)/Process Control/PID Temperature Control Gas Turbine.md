FUNCTION_BLOCK FB_GasTurbineTempPID
VAR_INPUT
    // Process Variable - Measured Temperature (°C)
    Temp_PV : REAL;

    // Setpoint - Target Temperature (°C)
    Temp_SP : REAL := 950.0;

    // PID Tuning Parameters
    Kp : REAL := 3.0;
    Ki : REAL := 0.7;
    Kd : REAL := 0.2;

    // Sampling Time (seconds, e.g., T#100ms = 0.1s)
    Sample_Time : REAL := 0.1;

    // Valve Position Limits (% open)
    Valve_Min : REAL := 0.0;
    Valve_Max : REAL := 100.0;
END_VAR

VAR_OUTPUT
    // Calculated output to the control valve
    Valve_Position : REAL;
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
Error := Temp_SP - Temp_PV;

// --- STEP 2: Update integral term with anti-windup ---
Integral_Limited := FALSE;

IF (Valve_Position > Valve_Max AND Error > 0) OR 
   (Valve_Position < Valve_Min AND Error < 0) THEN
    Integral_Limited := TRUE;
ELSE
    Integral := Integral + Error * Sample_Time;
END_IF;

// --- STEP 3: Compute derivative ---
Derivative := (Error - Prev_Error) / Sample_Time;

// --- STEP 4: Apply PID formula ---
Valve_Position := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

// --- STEP 5: Clamp output to valve limits ---
IF Valve_Position > Valve_Max THEN
    Valve_Position := Valve_Max;
ELSIF Valve_Position < Valve_Min THEN
    Valve_Position := Valve_Min;
END_IF;

// --- STEP 6: Save current error for next cycle ---
Prev_Error := Error;

PROGRAM PLC_PRG
VAR
    TurbineCtrl : FB_GasTurbineTempPID;

    // Simulated Inputs
    CurrentTemp : REAL := 948.5; // °C
    ValveCommand : REAL;
END_VAR

// Call the function block
TurbineCtrl(
    Temp_PV := CurrentTemp,
    Temp_SP := 950.0,
    Kp := 3.0,
    Ki := 0.7,
    Kd := 0.2,
    Sample_Time := 0.1,

    Valve_Position => ValveCommand
);
