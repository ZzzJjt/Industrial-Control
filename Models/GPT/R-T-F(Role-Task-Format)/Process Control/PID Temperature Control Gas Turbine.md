VAR
    // Inputs
    Temp_PV           : REAL;             // Measured turbine temperature (°C)
    Temp_SP           : REAL := 950.0;    // Desired turbine temperature setpoint (°C)

    // PID tuning parameters
    Kp                : REAL := 3.0;       // Proportional gain
    Ki                : REAL := 0.7;       // Integral gain
    Kd                : REAL := 0.2;       // Derivative gain

    // PID internal variables
    Error             : REAL;              // Current error
    Prev_Error        : REAL := 0.0;       // Previous error
    Integral          : REAL := 0.0;       // Accumulated integral
    Derivative        : REAL;              // Rate of change of error
    Valve_Position    : REAL;              // Output command for inlet valve (% open)

    // Output limits
    Valve_Min         : REAL := 0.0;       // Minimum valve opening (%)
    Valve_Max         : REAL := 100.0;     // Maximum valve opening (%)

    // Sampling time
    DeltaT            : REAL := 0.1;       // 100 ms loop time
END_VAR

// PID calculation (executed every 100 ms)
Error := Temp_SP - Temp_PV;
Integral := Integral + (Error * DeltaT);
Derivative := (Error - Prev_Error) / DeltaT;

Valve_Position := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

// Clamp the output to safe valve range
IF Valve_Position > Valve_Max THEN
    Valve_Position := Valve_Max;
ELSIF Valve_Position < Valve_Min THEN
    Valve_Position := Valve_Min;
END_IF

// Store the current error for the next derivative calculation
Prev_Error := Error;
