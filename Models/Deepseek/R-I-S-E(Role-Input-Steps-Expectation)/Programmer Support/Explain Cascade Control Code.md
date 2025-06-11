FUNCTION_BLOCK FB_CascadeControl_PressureFlow
VAR_INPUT
    // Primary Loop Inputs (Pressure Control)
    PV1 : REAL;              // Measured vessel pressure
    SP1 : REAL := 100.0;     // Pressure setpoint
    Kp1 : REAL := 2.5;       // Proportional gain (outer loop)
    Ki1 : REAL := 0.6;       // Integral gain
    Kd1 : REAL := 0.3;       // Derivative gain

    // Secondary Loop Inputs (Flow Control)
    PV2 : REAL;              // Measured flow rate
    Kp2 : REAL := 3.0;       // Proportional gain (inner loop)
    Ki2 : REAL := 0.8;       // Integral gain
    Kd2 : REAL := 0.1;       // Derivative gain

    // Common Settings
    dt : REAL := 0.1;        // Sample time in seconds (e.g., 100 ms)
    Enable_Control : BOOL := TRUE;
END_VAR

VAR_OUTPUT
    // Output signals
    OP1 : REAL;              // Output from outer loop (flow setpoint)
    OP2 : REAL;              // Final output to valve (% open)

    // Internal diagnostics
    SP2 : REAL;              // Flow setpoint passed to inner loop
    Error1 : REAL;           // Pressure error
    Error2 : REAL;           // Flow error
    In_Control : BOOL;
END_VAR

VAR
    // Outer loop variables (Pressure)
    Integral1 : REAL := 0.0;
    Derivative1 : REAL := 0.0;
    Prev_Error1 : REAL := 0.0;

    // Inner loop variables (Flow)
    Integral2 : REAL := 0.0;
    Derivative2 : REAL := 0.0;
    Prev_Error2 : REAL := 0.0;

    // Anti-windup flags
    Integral1_Limited : BOOL := FALSE;
    Integral2_Limited : BOOL := FALSE;

    // Output limits
    OP_Min : REAL := 0.0;
    OP_Max : REAL := 100.0;
END_VAR
