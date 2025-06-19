FUNCTION_BLOCK FB_PID_Controller
VAR_INPUT
    Setpoint        : REAL; // Desired value
    ProcessVariable : REAL; // Measured value
    Kp              : REAL; // Proportional gain
    Ki              : REAL; // Integral gain
    Kd              : REAL; // Derivative gain
    DeltaT          : REAL; // Time step (s)
    Enable          : BOOL; // Enable control
    Reset           : BOOL; // Reset controller
    MinOutput       : REAL; // Lower output limit
    MaxOutput       : REAL; // Upper output limit
END_VAR

VAR_OUTPUT
    Output          : REAL; // Controller output
    Error           : REAL; // Current error
    Integral        : REAL; // Integral term (for monitoring)
    Derivative      : REAL; // Derivative term (for monitoring)
END_VAR

VAR
    PrevError       : REAL := 0.0;  // Previous cycle error
    PrevPV          : REAL := 0.0;  // Previous process variable
    Windup          : BOOL := FALSE; // Anti-windup flag
END_VAR

// Reset logic
IF Reset THEN
    Integral := 0.0;
    Output := 0.0;
    PrevError := 0.0;
    PrevPV := 0.0;
    Windup := FALSE;
ELSE
    IF Enable THEN
        // Error calculation
        Error := Setpoint - ProcessVariable;

        // Integral term with anti-windup
        IF NOT Windup THEN
            Integral := Integral + Ki * Error * DeltaT;
        END_IF;

        // Derivative term using process variable
        Derivative := (ProcessVariable - PrevPV) / DeltaT;

        // PID output calculation
        Output := (Kp * Error) + Integral - (Kd * Derivative);

        // Output clamping with windup protection
        IF Output > MaxOutput THEN
            Output := MaxOutput;
            Windup := TRUE;
        ELSIF Output < MinOutput THEN
            Output := MinOutput;
            Windup := TRUE;
        ELSE
            Windup := FALSE;
        END_IF;

        // Store for next cycle
        PrevPV := ProcessVariable;
        PrevError := Error;
    END_IF;
END_IF;
