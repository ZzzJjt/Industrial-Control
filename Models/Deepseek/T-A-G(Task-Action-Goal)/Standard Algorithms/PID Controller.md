FUNCTION_BLOCK PIDControllerFunctionBlock
VAR_INPUT
    Setpoint : REAL;            // Desired setpoint value
    ProcessVariable : REAL;     // Current process variable value
    Kp : REAL;                  // Proportional gain
    Ki : REAL;                  // Integral gain
    Kd : REAL;                  // Derivative gain
    MinOutput : REAL;           // Minimum output limit
    MaxOutput : REAL;           // Maximum output limit
    Enable : BOOL;              // Enable signal to start control
    Reset : BOOL;               // Reset signal to reinitialize states
END_VAR

VAR_OUTPUT
    ControlOutput : REAL;       // Calculated control output
END_VAR

VAR
    Error : REAL := 0.0;        // Current error (Setpoint - ProcessVariable)
    PreviousError : REAL := 0.0;// Previous error for derivative calculation
    IntegralTerm : REAL := 0.0; // Accumulated integral term
    DerivativeTerm : REAL := 0.0; // Derivative term
    DeltaT : TIME := T#0s;      // Sample time (scan cycle time)
    LastTime : TIME := T#0s;    // Time of the last scan cycle
    OutputSum : REAL := 0.0;    // Sum of the output terms
END_VAR

METHOD Execute;
BEGIN
    // Initialize the control output
    ControlOutput := 0.0;

    // Check if the controller is enabled
    IF Enable THEN
        // Calculate the sample time (delta time)
        DeltaT := TONF(T#0s) - LastTime;
        LastTime := TONF(T#0s);

        // Compute the control error
        Error := Setpoint - ProcessVariable;

        // Accumulate the integral term with anti-windup
        IntegralTerm := IntegralTerm + Error * Ki * DeltaT / 1000.0; // Convert ms to s

        // Anti-windup: Clamp the integral term within output limits
        IF IntegralTerm > MaxOutput THEN
            IntegralTerm := MaxOutput;
        ELSIF IntegralTerm < MinOutput THEN
            IntegralTerm := MinOutput;
        END_IF;

        // Calculate the derivative term
        DerivativeTerm := Kd * (Error - PreviousError) / DeltaT * 1000.0; // Convert s to ms

        // Compute the control output
        OutputSum := Kp * Error + IntegralTerm + DerivativeTerm;

        // Apply output limits
        IF OutputSum > MaxOutput THEN
            ControlOutput := MaxOutput;
        ELSIF OutputSum < MinOutput THEN
            ControlOutput := MinOutput;
        ELSE
            ControlOutput := OutputSum;
        END_IF;

        // Update the previous error
        PreviousError := Error;
    ELSE
        // If not enabled, reset internal states
        IntegralTerm := 0.0;
        PreviousError := 0.0;
        OutputSum := 0.0;
        ControlOutput := 0.0;
    END_IF;

    // Reset internal states if Reset = TRUE
    IF Reset THEN
        IntegralTerm := 0.0;
        PreviousError := 0.0;
        OutputSum := 0.0;
        ControlOutput := 0.0;
    END_IF;
END_METHOD



FUNCTION_BLOCK PIDControllerFunctionBlock
VAR_INPUT
    Setpoint : REAL;            // Desired setpoint value
    ProcessVariable : REAL;     // Current process variable value
    Kp : REAL;                  // Proportional gain
    Ki : REAL;                  // Integral gain
    Kd : REAL;                  // Derivative gain
    MinOutput : REAL;           // Minimum output limit
    MaxOutput : REAL;           // Maximum output limit
    Enable : BOOL;              // Enable signal to start control
    Reset : BOOL;               // Reset signal to reinitialize states
END_VAR

VAR_OUTPUT
    ControlOutput : REAL;       // Calculated control output
END_VAR

VAR
    Error : REAL := 0.0;        // Current error (Setpoint - ProcessVariable)
    PreviousError : REAL := 0.0;// Previous error for derivative calculation
    IntegralTerm : REAL := 0.0; // Accumulated integral term
    DerivativeTerm : REAL := 0.0; // Derivative term
    DeltaT : TIME := T#0s;      // Sample time (scan cycle time)
    LastTime : TIME := T#0s;    // Time of the last scan cycle
    OutputSum : REAL := 0.0;    // Sum of the output terms
END_VAR

METHOD Execute;
BEGIN
    // Initialize the control output
    ControlOutput := 0.0;

    // Check if the controller is enabled
    IF Enable THEN
        // Calculate the sample time (delta time)
        DeltaT := TONF(T#0s) - LastTime;
        LastTime := TONF(T#0s);

        // Compute the control error
        Error := Setpoint - ProcessVariable;

        // Accumulate the integral term with anti-windup
        IntegralTerm := IntegralTerm + Error * Ki * DeltaT / 1000.0; // Convert ms to s

        // Anti-windup: Clamp the integral term within output limits
        IF IntegralTerm > MaxOutput THEN
            IntegralTerm := MaxOutput;
        ELSIF IntegralTerm < MinOutput THEN
            IntegralTerm := MinOutput;
        END_IF;

        // Calculate the derivative term
        DerivativeTerm := Kd * (Error - PreviousError) / DeltaT * 1000.0; // Convert s to ms

        // Compute the control output
        OutputSum := Kp * Error + IntegralTerm + DerivativeTerm;

        // Apply output limits
        IF OutputSum > MaxOutput THEN
            ControlOutput := MaxOutput;
        ELSIF OutputSum < MinOutput THEN
            ControlOutput := MinOutput;
        ELSE
            ControlOutput := OutputSum;
        END_IF;

        // Update the previous error
        PreviousError := Error;
    ELSE
        // If not enabled, reset internal states
        IntegralTerm := 0.0;
        PreviousError := 0.0;
        OutputSum := 0.0;
        ControlOutput := 0.0;
    END_IF;

    // Reset internal states if Reset = TRUE
    IF Reset THEN
        IntegralTerm := 0.0;
        PreviousError := 0.0;
        OutputSum := 0.0;
        ControlOutput := 0.0;
    END_IF;
END_METHOD



