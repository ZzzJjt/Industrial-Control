```
FUNCTION_BLOCK PIDController
VAR_INPUT
    pv : REAL; // Process variable (actual value)
    sp : REAL; // Setpoint (desired value)
    kp : REAL; // Proportional gain
    ki : REAL; // Integral gain
    kd : REAL; // Derivative gain
    ts : TIME; // Sampling time
END_VAR
VAR_OUTPUT
    output : REAL; // PID output
END_VAR
VAR
    err : REAL; // Error (difference between setpoint and process variable)
    intErr : REAL; // Integral of the error
    derErr : REAL; // Derivative of the error
    prevErr : REAL; // Previous error
    prevTs : TIME; // Previous sampling time
    antiWindup : REAL; // Anti-windup term
    integralTerm : REAL; // Integral term
    derivativeTerm : REAL; // Derivative term
    proportionalTerm : REAL; // Proportional term
END_VAR

// Initialize variables on the first call
IF prevTs = T#0s THEN
    prevTs := ts;
    prevErr := sp - pv;
    intErr := 0.0;
    derErr := 0.0;
END_IF;

// Calculate error
err := sp - pv;

// Calculate integral term
integralTerm := intErr * ki * TIME_TO_NUM(ts);

// Calculate derivative term
derErr := (err - prevErr) / NUM_TO_TIME(TIME_TO_NUM(ts) - TIME_TO_NUM(prevTs));
derErr := derErr * kd;

// Calculate proportional term
proportionalTerm := err * kp;

// Update integral error
intErr := intErr + err * TIME_TO_NUM(ts);

// Anti-windup (optional): prevent integral windup if output is saturated
antiWindup := IF output > MAX_OUTPUT OR output < MIN_OUTPUT THEN
                -err * ki * TIME_TO_NUM(ts);
              ELSE
                0.0;
              END_IF;

// Calculate PID output
output := proportionalTerm + integralTerm + derErr + antiWindup;

// Store current values for the next iteration
prevErr := err;
prevTs := ts;

RETURN;

END_FUNCTION_BLOCK

PROGRAM ExampleProgram
VAR
    processVar : REAL := 0.0; // Process variable (actual value)
    setPoint : REAL := 100.0; // Setpoint (desired value)
    pGain : REAL := 1.0; // Proportional gain
    iGain : REAL := 0.1; // Integral gain
    dGain : REAL := 0.01; // Derivative gain
    sampleTime : TIME := T#1s; // Sampling time
    pidOutput : REAL; // PID output
BEGIN
    // Call the PIDController function block
    PIDController(pv:=processVar, sp:=setPoint, kp:=pGain, ki:=iGain, kd:=dGain, ts:=sampleTime, output:=pidOutput);
    
    // Optionally, log or use the PID output
END_PROGRAM
```
