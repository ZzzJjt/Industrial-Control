FUNCTION_BLOCK FB_LevelControlPID
VAR_INPUT
    // Process Variable - Current Level (%)
    Level_PV : REAL;
    
    // Setpoint - Desired Level (%)
    Level_SP : REAL := 60.0;
    
    // PID Parameters
    Kp : REAL := 1.5;     // Proportional gain
    Ki : REAL := 0.4;     // Integral gain
    Kd : REAL := 0.2;     // Derivative gain
    
    // Sample Time (seconds)
    Sample_Time : REAL := 0.1;
    
    // Valve Position Limits
    Valve_Min : REAL := 0.0;
    Valve_Max : REAL := 100.0;
END_VAR

VAR_OUTPUT
    // Output - Valve Position (%)
    Valve_Position : REAL;
END_VAR

VAR
    // Internal Variables for PID Calculation
    Error : REAL;
    Prev_Error : REAL := 0.0;
    Integral : REAL := 0.0;
    Derivative : REAL;
    
    // Flags or additional internal states can be added here if needed
END_VAR

// Calculate the error between setpoint and process variable
Error := Level_SP - Level_PV;

// Update integral term with anti-windup prevention
IF (Valve_Position > Valve_Max AND Error > 0) OR (Valve_Position < Valve_Min AND Error < 0) THEN
    // Do not update integral if output is saturated
ELSE
    Integral := Integral + Error * Sample_Time;
END_IF;

// Compute derivative based on change in error
Derivative := (Error - Prev_Error) / Sample_Time;

// Calculate PID output
Valve_Position := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

// Clamp the output to ensure it stays within safe operational limits
IF Valve_Position > Valve_Max THEN
    Valve_Position := Valve_Max;
ELSIF Valve_Position < Valve_Min THEN
    Valve_Position := Valve_Min;
END_IF;

// Store current error for use in next cycle
Prev_Error := Error;

PROGRAM MainLoop
VAR
    LevelCtrl : FB_LevelControlPID;
    MeasuredLevel : REAL; // Simulated or actual input from sensor
    ValveCmd : REAL;      // Command sent to valve actuator
END_VAR

// Assume MeasuredLevel is updated elsewhere in the program
MeasuredLevel := ...; // Your method of getting the actual level measurement

LevelCtrl(Level_PV := MeasuredLevel,
          Level_SP := 60.0,
          Kp := 1.5,
          Ki := 0.4,
          Kd := 0.2,
          Sample_Time := 0.1,
          Valve_Min := 0.0,
          Valve_Max := 100.0,
          Valve_Position => ValveCmd);

// Now, ValveCmd contains the calculated valve position command
