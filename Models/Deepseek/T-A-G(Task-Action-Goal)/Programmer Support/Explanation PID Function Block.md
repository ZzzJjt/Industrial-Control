PROGRAM MainProgram
VAR
    // Define PID instance
    MyPID : FB_PID;

    // Inputs
    SP : REAL := 75.0; // Desired setpoint (e.g., temperature)
    PV : REAL := 65.0; // Measured process variable (e.g., current temperature)
    Kp : REAL := 1.0;  // Proportional gain
    Ki : REAL := 0.1;  // Integral gain
    Kd : REAL := 0.01; // Derivative gain
    MANUAL : BOOL := FALSE; // Manual mode flag
    LIMIT_HI : REAL := 100.0; // Upper output limit
    LIMIT_LO : REAL := 0.0;  // Lower output limit
    RESET : BOOL := FALSE;   // Reset integral term flag

    // Outputs
    OUT : REAL; // Control signal to actuator
    ERROR : REAL; // Error between setpoint and process variable
    SATURATED : BOOL; // Saturation status
    INTEGRAL : REAL; // Integral term value
    DERIVATIVE : REAL; // Derivative term value
END_VAR

// Call the PID function block
MyPID(
    SP := SP,
    PV := PV,
    Kp := Kp,
    Ki := Ki,
    Kd := Kd,
    MANUAL := MANUAL,
    LIMIT_HI := LIMIT_HI,
    LIMIT_LO := LIMIT_LO,
    RESET := RESET,
    OUT => OUT,
    ERROR => ERROR,
    SATURATED => SATURATED,
    INTEGRAL => INTEGRAL,
    DERIVATIVE => DERIVATIVE
);

// Additional logic can be added here to simulate changes in PV or adjust parameters



