FUNCTION_BLOCK FB_PIDController
VAR_INPUT
    Setpoint      : REAL;  // Desired value
    ProcessVariable : REAL;  // Measured value
    Kp            : REAL;  // Proportional gain
    Ki            : REAL;  // Integral gain
    Kd            : REAL;  // Derivative gain
    MinOutput     : REAL;  // Minimum output limit
    MaxOutput     : REAL;  // Maximum output limit
    Enable        : BOOL;  // Enable calculation
    Reset         : BOOL;  // Reset internal states
    DeltaT        : REAL;  // Time between calls in seconds
END_VAR

VAR_OUTPUT
    ControlOutput : REAL;  // Output signal
    Error         : REAL;  // Current error
    IntegralTerm  : REAL;  // Integral component
    DerivativeTerm: REAL;  // Derivative component
END_VAR

VAR
    LastError     : REAL := 0.0;
    IntegralAcc   : REAL := 0.0;
END_VAR

IF Reset THEN
    IntegralAcc := 0.0;
    LastError := 0.0;
    ControlOutput := 0.0;
END_IF

IF Enable THEN
    // Error calculation
    Error := Setpoint - ProcessVariable;

    // Integral term with anti-windup clamp
    IntegralAcc := IntegralAcc + Error * DeltaT;
    IF IntegralAcc * Ki > MaxOutput THEN
        IntegralAcc := MaxOutput / Ki;
    ELSIF IntegralAcc * Ki < MinOutput THEN
        IntegralAcc := MinOutput / Ki;
    END_IF
    IntegralTerm := IntegralAcc;

    // Derivative term
    DerivativeTerm := (Error - LastError) / DeltaT;

    // PID Output
    ControlOutput := Kp * Error + Ki * IntegralTerm + Kd * DerivativeTerm;

    // Clamp output
    IF ControlOutput > MaxOutput THEN
        ControlOutput := MaxOutput;
    ELSIF ControlOutput < MinOutput THEN
        ControlOutput := MinOutput;
    END_IF

    // Update last error
    LastError := Error;
END_IF
