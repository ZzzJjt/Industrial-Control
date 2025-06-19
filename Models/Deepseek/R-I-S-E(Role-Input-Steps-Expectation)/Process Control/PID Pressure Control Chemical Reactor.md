FUNCTION_BLOCK FB_ReactorPressurePID
VAR_INPUT
    // Process Variable - Measured Pressure (bar)
    Pressure_PV : REAL;

    // Setpoint - Target Pressure (bar)
    Pressure_SP : REAL := 5.0;

    // Tuning Parameters
    Kp : REAL := 2.0;
    Ki : REAL := 0.8;
    Kd : REAL := 0.3;

    // Sampling Time (seconds, e.g., T#100ms = 0.1s)
    Sample_Time : REAL := 0.1;

    // Valve Position Limits (% open)
    Valve_Min : REAL := 0.0;
    Valve_Max : REAL := 100.0;
END_VAR

VAR_OUTPUT
    // Calculated output to the control valve
    Valve_Output : REAL;
END_VAR

VAR
    // Internal Variables
    Error : REAL;
    Prev_Error : REAL := 0.0;
    Integral : REAL := 0.0;
    Derivative : REAL := 0.0;

    // Anti-windup flag
    Integral_Limited : BOOL := FALSE;
END_VAR

// --- STEP 1: Calculate error ---
Error := Pressure_SP - Pressure_PV;

// --- STEP 2: Update integral term with anti-windup ---
Integral_Limited := FALSE;

IF (Valve_Output >= Valve_Max AND Error > 0) OR 
   (Valve_Output <= Valve_Min AND Error < 0) THEN
    Integral_Limited := TRUE;
ELSE
    Integral := Integral + Error * Sample_Time;
END_IF;

// --- STEP 3: Compute derivative ---
Derivative := (Error - Prev_Error) / Sample_Time;

// --- STEP 4: Apply PID formula ---
Valve_Output := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

// --- STEP 5: Clamp output to valve limits ---
IF Valve_Output > Valve_Max THEN
    Valve_Output := Valve_Max;
ELSIF Valve_Output < Valve_Min THEN
    Valve_Output := Valve_Min;
END_IF;

// --- STEP 6: Save current error for next cycle ---
Prev_Error := Error;

PROGRAM PLC_PRG
VAR
    ReactorCtrl : FB_ReactorPressurePID;

    // Simulated Inputs
    CurrentPressure : REAL := 4.9; // Bar
    ValveCommand : REAL;
END_VAR

// Call the function block
ReactorCtrl(
    Pressure_PV := CurrentPressure,
    Pressure_SP := 5.0,
    Kp := 2.0,
    Ki := 0.8,
    Kd := 0.3,
    Sample_Time := 0.1,

    Valve_Output => ValveCommand
);
