FUNCTION_BLOCK FB_PIDController
VAR_INPUT
    Setpoint : REAL; // Desired setpoint value
    ProcessVariable : REAL; // Current process variable value
    Kp : REAL; // Proportional gain
    Ki : REAL; // Integral gain
    Kd : REAL; // Derivative gain
    MinOutput : REAL; // Minimum output limit
    MaxOutput : REAL; // Maximum output limit
    Enable : BOOL; // Enable flag to start the PID control
    Reset : BOOL; // Reset flag to reinitialize internal states
END_VAR

VAR_OUTPUT
    ControlOutput : REAL; // Calculated control output
    Error : REAL; // Current error (Setpoint - ProcessVariable)
    IntegralTerm : REAL; // Integral term of the PID controller
    DerivativeTerm : REAL; // Derivative term of the PID controller
END_VAR

VAR
    LastProcessVariable : REAL := 0.0; // Previous process variable value for derivative calculation
    IntegralSum : REAL := 0.0; // Sum of the integral term
    LastError : REAL := 0.0; // Previous error value for derivative calculation
    DeltaTime : TIME := T#1s; // Time interval between scan cycles (adjust as needed)
    LastTime : TIME := T#0s; // Time of the last scan cycle
END_VAR

// Method to initialize the PID controller states
METHOD Initialize(this : REFERENCE TO FB_PIDController)
BEGIN
    this.LastProcessVariable := this.ProcessVariable;
    this.IntegralSum := 0.0;
    this.LastError := 0.0;
    this.ControlOutput := 0.0;
    this.Error := 0.0;
    this.IntegralTerm := 0.0;
    this.DerivativeTerm := 0.0;
END_METHOD

// Main method to execute the PID control logic
METHOD Execute(this : REFERENCE TO FB_PIDController) : BOOL
BEGIN
    VAR
        CurrentTime : TIME;
        ElapsedTime : TIME;
        ElapsedSeconds : REAL;
        PTerm : REAL;
        ITerm : REAL;
        DTerm : REAL;
        NewControlOutput : REAL;
    END_VAR

    // Get current time
    CurrentTime := TP();

    // Calculate elapsed time since last execution
    ElapsedTime := CurrentTime - this.LastTime;
    ElapsedSeconds := REAL(ElapsedTime) / 1000.0;

    // Update last time
    this.LastTime := CurrentTime;

    // Validate inputs
    IF NOT this.Enable THEN
        this.ControlOutput := 0.0;
        RETURN TRUE;
    END_IF;

    // Reset the filter if the Reset flag is set
    IF this.Reset THEN
        this.Initialize();
        RETURN TRUE;
    END_IF;

    // Calculate error
    this.Error := this.Setpoint - this.ProcessVariable;

    // Proportional term
    PTerm := this.Kp * this.Error;

    // Integral term with anti-windup
    IF this.Error < this.MaxOutput AND this.Error > this.MinOutput THEN
        this.IntegralSum := this.IntegralSum + this.Error * ElapsedSeconds;
    END_IF;
    ITerm := this.Ki * this.IntegralSum;

    // Derivative term
    DTerm := this.Kd * ((this.ProcessVariable - this.LastProcessVariable) / ElapsedSeconds);

    // Calculate new control output
    NewControlOutput := PTerm + ITerm - DTerm;

    // Clamp the control output within specified limits
    IF NewControlOutput > this.MaxOutput THEN
        this.ControlOutput := this.MaxOutput;
    ELSIF NewControlOutput < this.MinOutput THEN
        this.ControlOutput := this.MinOutput;
    ELSE
        this.ControlOutput := NewControlOutput;
    END_IF;

    // Store current values for next iteration
    this.LastProcessVariable := this.ProcessVariable;
    this.LastError := this.Error;

    // Optional: Output individual terms for monitoring
    this.IntegralTerm := ITerm;
    this.DerivativeTerm := DTerm;

    RETURN TRUE;
END_METHOD

// Main execution method
METHOD MAIN(this : REFERENCE TO FB_PIDController) : BOOL
BEGIN
    RETURN this.Execute();
END_METHOD


PROGRAM MainProgram
VAR
    PIDControllerFB : FB_PIDController;
    Setpoint : REAL := 100.0; // Desired setpoint
    ProcessVariable : REAL := 50.0; // Current process variable
    Kp : REAL := 1.0; // Proportional gain
    Ki : REAL := 0.1; // Integral gain
    Kd : REAL := 0.01; // Derivative gain
    MinOutput : REAL := -100.0; // Minimum output limit
    MaxOutput : REAL := 100.0; // Maximum output limit
    Enable : BOOL := TRUE; // Enable flag
    Reset : BOOL := FALSE; // Reset flag
    ControlOutput : REAL;
    Error : REAL;
    IntegralTerm : REAL;
    DerivativeTerm : REAL;
END_VAR

// Set the inputs for the PID controller
PIDControllerFB.Setpoint := Setpoint;
PIDControllerFB.ProcessVariable := ProcessVariable;
PIDControllerFB.Kp := Kp;
PIDControllerFB.Ki := Ki;
PIDControllerFB.Kd := Kd;
PIDControllerFB.MinOutput := MinOutput;
PIDControllerFB.MaxOutput := MaxOutput;
PIDControllerFB.Enable := Enable;
PIDControllerFB.Reset := Reset;

// Call the PID controller function block
PIDControllerFB();

// Get the control output and other terms
ControlOutput := PIDControllerFB.ControlOutput;
Error := PIDControllerFB.Error;
IntegralTerm := PIDControllerFB.IntegralTerm;
DerivativeTerm := PIDControllerFB.DerivativeTerm;

// Output the results
// For demonstration purposes, assume there's a way to display or log the results
// Display(ControlOutput); // Hypothetical display function
// Display(Error); // Hypothetical display function
// Display(IntegralTerm); // Hypothetical display function
// Display(DerivativeTerm); // Hypothetical display function
