VAR
    // Inputs
    Pressure_PV     : REAL;              // Measured reactor pressure (bar)
    Pressure_SP     : REAL := 5.0;       // Desired pressure setpoint (bar)

    // PID gains
    Kp              : REAL := 2.0;       // Proportional gain
    Ki              : REAL := 0.8;       // Integral gain
    Kd              : REAL := 0.3;       // Derivative gain

    // Internal PID variables
    Error           : REAL;              // Current error
    Prev_Error      : REAL := 0.0;       // Previous error
    Integral        : REAL := 0.0;       // Accumulated integral term
    Derivative      : REAL;              // Rate of change of error
    Valve_Output    : REAL;              // Output command to pressure control valve (0–100%)

    // Safety constraints
    Valve_Min       : REAL := 0.0;       // Minimum valve opening (%)
    Valve_Max       : REAL := 100.0;     // Maximum valve opening (%)

    // Sampling interval
    DeltaT          : REAL := 0.1;       // 100 ms time step
END_VAR

// PID control logic (executed every 100 ms)
Error := Pressure_SP - Pressure_PV;
Integral := Integral + (Error * DeltaT);
Derivative := (Error - Prev_Error) / DeltaT;

Valve_Output := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

// Clamp output to safe valve range
IF Valve_Output > Valve_Max THEN
    Valve_Output := Valve_Max;
ELSIF Valve_Output < Valve_Min THEN
    Valve_Output := Valve_Min;
END_IF

// Store error for next derivative calculation
Prev_Error := Error;
