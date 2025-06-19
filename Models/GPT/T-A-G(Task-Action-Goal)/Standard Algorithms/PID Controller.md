FUNCTION_BLOCK FB_PIDController
VAR_INPUT
    Setpoint       : REAL;     // Desired target value
    ProcessVariable: REAL;     // Measured process value
    Kp             : REAL;     // Proportional gain
    Ki             : REAL;     // Integral gain
    Kd             : REAL;     // Derivative gain
    DeltaT         : REAL;     // Time step in seconds
    MinOutput      : REAL;     // Lower bound for output
    MaxOutput      : REAL;     // Upper bound for output
    Enable         : BOOL;     // Enable control
    Reset          : BOOL;     // Reset internal states
END_VAR

VAR_OUTPUT
    ControlOutput  : REAL;     // Final PID output
    Error          : REAL;     // Current error value
END_VAR

VAR
    Integral       : REAL := 0.0;      // Integral accumulator
    PrevError      : REAL := 0.0;      // Error from previous cycle
    Derivative     : REAL := 0.0;      // Derivative term
    UnclampedOutput: REAL := 0.0;      // Raw PID output (before clamping)
END_VAR

// ========== RESET LOGIC ==========
IF Reset THEN
    Integral := 0.0;
    PrevError := 0.0;
    ControlOutput := 0.0;
    UnclampedOutput := 0.0;
    Derivative := 0.0;
    RETURN;
END_IF

// ========== PID COMPUTATION ==========
IF Enable THEN
    // Compute current error
    Error := Setpoint - ProcessVariable;

    // Proportional term
    UnclampedOutput := Kp * Error;

    // Integral term with anti-windup
    Integral := Integral + (Ki * Error * DeltaT);
    // Pre-clamp to avoid windup
    IF Integral > MaxOutput THEN
        Integral := MaxOutput;
    ELSIF Integral < MinOutput THEN
        Integral := MinOutput;
    END_IF;

    // Derivative term (based on process variable change)
    Derivative := (Kd * (Error - PrevError)) / DeltaT;
    PrevError := Error;

    // Sum all terms
    UnclampedOutput := UnclampedOutput + Integral + Derivative;

    // Apply output clamping
    IF UnclampedOutput > MaxOutput THEN
        ControlOutput := MaxOutput;
    ELSIF UnclampedOutput < MinOutput THEN
        ControlOutput := MinOutput;
    ELSE
        ControlOutput := UnclampedOutput;
    END_IF;
ELSE
    // If not enabled, hold last output
    ControlOutput := ControlOutput;
END_IF
