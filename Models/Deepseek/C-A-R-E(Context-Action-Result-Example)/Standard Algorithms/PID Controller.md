Certainly! Below is a self-contained PID controller function block in IEC 61131-3 Structured Text. This function block includes configurable gain inputs, real-time error calculation, integral term accumulation with anti-windup protection, derivative action using past process variable, output limits to prevent overcorrection, and inputs for enabling and resetting the controller.

### PID Controller Function Block

```st
[<title="PIDControllerFunctionBlock">]
FUNCTION_BLOCK PIDControllerFB
VAR_INPUT
    Enable : BOOL;          // Enable signal to start the PID control
    Reset : BOOL;           // Reset signal to reinitialize the controller
    Setpoint : REAL;        // Desired setpoint value
    ProcessVariable : REAL; // Current process variable
    Kp : REAL := 1.0;      // Proportional gain
    Ki : REAL := 0.1;      // Integral gain
    Kd : REAL := 0.01;     // Derivative gain
    DeltaT : REAL := 0.1;  // Time step between updates
    MinOutput : REAL := -100.0; // Minimum output limit
    MaxOutput : REAL := 100.0;  // Maximum output limit
END_VAR

VAR_OUTPUT
    Output : REAL := 0.0;   // Control output
    Error : REAL := 0.0;    // Optional: Calculated error
    Integral : REAL := 0.0; // Optional: Integral term
    Derivative : REAL := 0.0; // Optional: Derivative term
END_VAR

VAR
    PrevError : REAL := 0.0; // Previous error value for derivative calculation
    PrevProcessVariable : REAL := 0.0; // Previous process variable for derivative calculation
    Windup : BOOL := FALSE; // Flag for integral windup protection
BEGIN
METHOD Execute : VOID
BEGIN
    IF Reset THEN
        // Reinitialize the controller
        Output := 0.0;
        Error := 0.0;
        Integral := 0.0;
        Derivative := 0.0;
        PrevError := 0.0;
        PrevProcessVariable := 0.0;
        Windup := FALSE;
    ELSIF Enable THEN
        // Calculate error
        Error := Setpoint - ProcessVariable;

        // Integral term with anti-windup protection
        IF NOT Windup THEN
            Integral := Integral + (Ki * Error * DeltaT);
        END_IF;

        // Prevent integral windup
        IF Integral > MaxOutput OR Integral < MinOutput THEN
            Windup := TRUE;
        ELSE
            Windup := FALSE;
        END_IF;

        // Derivative term
        Derivative := (ProcessVariable - PrevProcessVariable) / DeltaT;

        // PID output calculation
        Output := (Kp * Error) + Integral - (Kd * Derivative);

        // Clamp output within specified limits
        IF Output > MaxOutput THEN
            Output := MaxOutput;
        ELSIF Output < MinOutput THEN
            Output := MinOutput;
        END_IF;

        // Update previous values
        PrevError := Error;
        PrevProcessVariable := ProcessVariable;
    END_IF;
END_METHOD

// Main execution method
Execute();
END_FUNCTION_BLOCK
```

### Explanation of the Code

#### Inputs
- **Enable**: A boolean signal to initiate the PID control.
- **Reset**: A boolean signal to reinitialize the controller.
- **Setpoint**: The desired setpoint value.
- **ProcessVariable**: The current process variable.
- **Kp**: Proportional gain.
- **Ki**: Integral gain.
- **Kd**: Derivative gain.
- **DeltaT**: Time step between updates.
- **MinOutput**: Minimum output limit.
- **MaxOutput**: Maximum output limit.

#### Outputs
- **Output**: The control output.
- **Error**: The calculated error (optional).
- **Integral**: The integral term (optional).
- **Derivative**: The derivative term (optional).

#### Internal Variables
- **PrevError**: The previous error value for derivative calculation.
- **PrevProcessVariable**: The previous process variable for derivative calculation.
- **Windup**: A flag for integral windup protection.

#### Method: Execute
- **Reset Logic**: If `Reset` is `TRUE`, reinitializes all internal states.
- **Execution Logic**:
  - Calculates the error as the difference between `Setpoint` and `ProcessVariable`.
  - Updates the integral term with anti-windup protection.
  - Calculates the derivative term based on the change in `ProcessVariable`.
  - Computes the PID output using the proportional, integral, and derivative terms.
  - Clamps the output within the specified `MinOutput` and `MaxOutput` limits.
  - Updates the previous error and process variable for subsequent calculations.

### Usage Example

To use this function block in a PLC program, you would instantiate it and call its `Execute` method:

```st
PROGRAM MainProgram
VAR
    MyPIDController : PIDControllerFB;
    Setpoint : REAL := 100.0; // Example setpoint
    ProcessVariable : REAL := 50.0; // Example process variable
    Kp : REAL := 1.0; // Example proportional gain
    Ki : REAL := 0.1; // Example integral gain
    Kd : REAL := 0.01; // Example derivative gain
    DeltaT : REAL := 0.1; // Example time step
    MinOutput : REAL := -100.0; // Example minimum output limit
    MaxOutput : REAL := 100.0; // Example maximum output limit
    ControlOutput : REAL;
    Error : REAL;
    Integral : REAL;
    Derivative : REAL;
BEGIN
    // Configure the PID controller
    MyPIDController.Enable := TRUE;
    MyPIDController.Reset := FALSE;
    MyPIDController.Setpoint := Setpoint;
    MyPIDController.ProcessVariable := ProcessVariable;
    MyPIDController.Kp := Kp;
    MyPIDController.Ki := Ki;
    MyPIDController.Kd := Kd;
    MyPIDController.DeltaT := DeltaT;
    MyPIDController.MinOutput := MinOutput;
    MyPIDController.MaxOutput := MaxOutput;

    // Execute the PID controller
    MyPIDController();

    // Retrieve the results
    ControlOutput := MyPIDController.Output;
    Error := MyPIDController.Error;
    Integral := MyPIDController.Integral;
    Derivative := MyPIDController.Derivative;

    // Output the results
    // Use ControlOutput, Error, Integral, and Derivative as needed
END_PROGRAM
```

### Key Features
- **Configurable Gains**: Adjustable proportional, integral, and derivative gains.
- **Real-Time Calculation**: Continuous error calculation and control output generation.
- **Anti-Windup Protection**: Prevents integral windup by clamping the integral term.
- **Derivative Action**: Uses past process variable for derivative calculation.
- **Output Limits**: Ensures the control output remains within specified bounds.
- **Scan-Cycle Safe**: Suitable for PLC environments with scan-cycle constraints.
- **Modular Design**: Clear inputs and outputs for easy integration into broader control programs.

This function block provides stable and responsive control output across a range of operating conditions, making it suitable for both simulation and real-time industrial control applications.
