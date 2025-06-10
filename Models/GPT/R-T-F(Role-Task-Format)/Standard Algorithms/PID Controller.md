FUNCTION_BLOCK FB_PIDController
VAR_INPUT
    Setpoint       : REAL;   // Desired value
    ProcessVariable: REAL;   // Measured process value
    Kp             : REAL;   // Proportional gain
    Ki             : REAL;   // Integral gain
    Kd             : REAL;   // Derivative gain
    MinOutput      : REAL;   // Lower clamp limit
    MaxOutput      : REAL;   // Upper clamp limit
    Enable         : BOOL;   // Enable execution
    Reset          : BOOL;   // Reset internal states
END_VAR

VAR_OUTPUT
    ControlOutput  : REAL;   // Final output
    Error          : REAL;   // Current control error (SP - PV)
    IntegralTerm   : REAL;   // Integral contribution
    DerivativeTerm : REAL;   // Derivative contribution
END_VAR

VAR
    PrevError      : REAL := 0.0;  // Previous error
    PrevPV         : REAL := 0.0;  // Previous process variable
    DeltaT         : REAL := 0.1;  // Time step (can be tuned externally)
    RawOutput      : REAL;
    Derivative     : REAL;
END_VAR

// === Reset block ===
IF Reset THEN
    IntegralTerm := 0.0;
    DerivativeTerm := 0.0;
    PrevError := 0.0;
    PrevPV := ProcessVariable;
    ControlOutput := 0.0;
END_IF

// === PID logic executes only if enabled ===
IF Enable THEN
    // Compute error
    Error := Setpoint - ProcessVariable;

    // Proportional term
    RawOutput := Kp * Error;

    // Integral term with anti-windup clamp
    IntegralTerm := IntegralTerm + (Ki * Error * DeltaT);
    IF IntegralTerm > MaxOutput THEN
        IntegralTerm := MaxOutput;
    ELSIF IntegralTerm < MinOutput THEN
        IntegralTerm := MinOutput;
    END_IF
    RawOutput := RawOutput + IntegralTerm;

    // Derivative term (based on PV rate of change)
    Derivative := (ProcessVariable - PrevPV) / DeltaT;
    DerivativeTerm := -Kd * Derivative;
    RawOutput := RawOutput + DerivativeTerm;

    // Clamp total output
    IF RawOutput > MaxOutput THEN
        ControlOutput := MaxOutput;
    ELSIF RawOutput < MinOutput THEN
        ControlOutput := MinOutput;
    ELSE
        ControlOutput := RawOutput;
    END_IF

    // Save previous values for next cycle
    PrevError := Error;
    PrevPV := ProcessVariable;
END_IF
