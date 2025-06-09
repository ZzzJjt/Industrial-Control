FUNCTION_BLOCK FB_PIDController
VAR_INPUT
    Setpoint       : REAL;    // Desired value
    ProcessVariable: REAL;    // Measured value
    Kp             : REAL;    // Proportional gain
    Ki             : REAL;    // Integral gain
    Kd             : REAL;    // Derivative gain
    MinOutput      : REAL;    // Lower output clamp
    MaxOutput      : REAL;    // Upper output clamp
    Enable         : BOOL;    // Controller active
    Reset          : BOOL;    // Reset integrator and state
    DeltaT         : REAL;    // Scan time [s]
END_VAR

VAR_OUTPUT
    ControlOutput  : REAL;    // Final controller output
    Error          : REAL;    // Tracking error
    IntegralTerm   : REAL;    // Accumulated integral term
    DerivativeTerm : REAL;    // Derivative term
END_VAR

VAR
    PrevError        : REAL := 0.0;
    PrevProcessValue : REAL := 0.0;
    RawOutput        : REAL;
    DerivativeRaw    : REAL;
END_VAR

IF Reset THEN
    // Reset internal states
    IntegralTerm := 0.0;
    PrevError := 0.0;
    PrevProcessValue := ProcessVariable;
    ControlOutput := 0.0;
    DerivativeTerm := 0.0;
ELSIF Enable THEN
    // Calculate error
    Error := Setpoint - ProcessVariable;

    // --- Proportional term ---
    RawOutput := Kp * Error;

    // --- Integral term with anti-windup ---
    IntegralTerm := IntegralTerm + (Ki * Error * DeltaT);

    // Clamp integral if output is beyond limits (anti-windup)
    IF IntegralTerm > MaxOutput THEN
        IntegralTerm := MaxOutput;
    ELSIF IntegralTerm < MinOutput THEN
        IntegralTerm := MinOutput;
    END_IF;

    // --- Derivative term (on measurement) ---
    DerivativeRaw := (ProcessVariable - PrevProcessValue) / DeltaT;
    DerivativeTerm := -Kd * DerivativeRaw;

    // --- Total PID output ---
    ControlOutput := RawOutput + IntegralTerm + DerivativeTerm;

    // Clamp final output
    IF ControlOutput > MaxOutput THEN
        ControlOutput := MaxOutput;
    ELSIF ControlOutput < MinOutput THEN
        ControlOutput := MinOutput;
    END_IF;

    // Store for next cycle
    PrevProcessValue := ProcessVariable;
END_IF

PID1 : FB_PIDController;

PID1.Setpoint := 75.0;
PID1.ProcessVariable := TempSensor;
PID1.Kp := 2.5;
PID1.Ki := 0.8;
PID1.Kd := 0.1;
PID1.MinOutput := 0.0;
PID1.MaxOutput := 100.0;
PID1.Enable := TRUE;
PID1.Reset := FALSE;
PID1.DeltaT := 0.1;

PID1(); // Execute controller

ActuatorOutput := PID1.ControlOutput;
