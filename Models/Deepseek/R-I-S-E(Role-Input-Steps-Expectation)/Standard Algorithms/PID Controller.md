FUNCTION_BLOCK PIDControllerFB
VAR_INPUT
    Setpoint : REAL;          // Desired setpoint value
    ProcessVariable : REAL;   // Current process variable
    Kp : REAL;                // Proportional gain
    Ki : REAL;                // Integral gain
    Kd : REAL;                // Derivative gain
    MinOutput : REAL;         // Minimum output limit
    MaxOutput : REAL;         // Maximum output limit
    Enable : BOOL;            // Enable control loop
    Reset : BOOL;             // Reset internal states
END_VAR

VAR_OUTPUT
    ControlOutput : REAL;     // Calculated control output
    Error : REAL;             // Control error
    IntegralTerm : REAL;      // Integral term
    DerivativeTerm : REAL;    // Derivative term
END_VAR

VAR
    LastProcessVariable : REAL := 0.0; // Previous process variable for derivative calculation
    IntegralSum : REAL := 0.0;         // Accumulated integral sum
    WindupGuardActive : BOOL := FALSE; // Flag to prevent windup
    DeltaT : REAL := 0.1;              // Sample time (adjust as needed)
    LastTime : TIME;                   // Time of last execution
    CurrentTime : TIME;                // Current time
END_VAR

METHOD Execute : VOID
BEGIN
    // Get current time
    CurrentTime := TONOT(T#0S);

    // Calculate delta time
    IF LastTime <> T#0S THEN
        DeltaT := (CurrentTime - LastTime) / 1000.0; // Convert to seconds
    END_IF;
    LastTime := CurrentTime;

    // Reset logic on rising edge of Reset signal
    IF Reset AND NOT EdgeDetector(Reset) THEN
        IntegralSum := 0.0;
        WindupGuardActive := FALSE;
        LastProcessVariable := ProcessVariable;
    END_IF;

    // Control loop execution
    IF Enable THEN
        // Calculate error
        Error := Setpoint - ProcessVariable;

        // Update integral term with anti-windup
        IF NOT WindupGuardActive THEN
            IntegralSum := IntegralSum + Error * DeltaT;
            IF ControlOutput >= MaxOutput OR ControlOutput <= MinOutput THEN
                WindupGuardActive := TRUE;
            END_IF;
        ELSE
            IF ControlOutput < MaxOutput AND ControlOutput > MinOutput THEN
                WindupGuardActive := FALSE;
            END_IF;
        END_IF;

        // Calculate derivative term
        DerivativeTerm := (ProcessVariable - LastProcessVariable) / DeltaT;

        // Compute PID output
        ControlOutput := Kp * Error + Ki * IntegralSum - Kd * DerivativeTerm;

        // Apply output clamping
        IF ControlOutput > MaxOutput THEN
            ControlOutput := MaxOutput;
        ELSIF ControlOutput < MinOutput THEN
            ControlOutput := MinOutput;
        END_IF;

        // Store current process variable for next derivative calculation
        LastProcessVariable := ProcessVariable;
    ELSE
        // Disable control loop: reset outputs
        ControlOutput := 0.0;
        Error := 0.0;
        IntegralTerm := 0.0;
        DerivativeTerm := 0.0;
    END_IF;
END_METHOD

// Helper method to detect rising edge of a boolean signal
FUNCTION EdgeDetector : BOOL
VAR_INPUT
    Signal : BOOL;
END_VAR

VAR_STATIC
    LastSignal : BOOL := FALSE;
END_VAR

BEGIN
    EdgeDetector := Signal AND NOT LastSignal;
    LastSignal := Signal;
END_FUNCTION

END_FUNCTION_BLOCK

PROGRAM MainProgram
VAR
    PIDFB : PIDControllerFB;
    Setpoint : REAL := 100.0;           // Desired setpoint
    ProcessVariable : REAL := 0.0;      // Current process variable
    Kp : REAL := 1.0;                   // Proportional gain
    Ki : REAL := 0.1;                   // Integral gain
    Kd : REAL := 0.01;                  // Derivative gain
    MinOutput : REAL := -100.0;         // Minimum output limit
    MaxOutput : REAL := 100.0;          // Maximum output limit
    Enable : BOOL := TRUE;              // Enable control loop
    Reset : BOOL := FALSE;              // Reset flag
    ControlOutput : REAL;
    Error : REAL;
    IntegralTerm : REAL;
    DerivativeTerm : REAL;
END_VAR

// Configure the function block with sample inputs
PIDFB(Setpoint := Setpoint,
      ProcessVariable := ProcessVariable,
      Kp := Kp,
      Ki := Ki,
      Kd := Kd,
      MinOutput := MinOutput,
      MaxOutput := MaxOutput,
      Enable := Enable,
      Reset := Reset);

// Call the function block
PIDFB.Execute();

// Use the outputs
ControlOutput := PIDFB.ControlOutput;
Error := PIDFB.Error;
IntegralTerm := PIDFB.IntegralTerm;
DerivativeTerm := PIDFB.DerivativeTerm;

// Example usage: Print or log the results
// Note: In a real PLC environment, you would typically use outputs to control other parts of the system
IF Enable THEN
    // Process the control output and other terms
ELSE
    // Handle disabled control loop
END_IF;
END_PROGRAM
