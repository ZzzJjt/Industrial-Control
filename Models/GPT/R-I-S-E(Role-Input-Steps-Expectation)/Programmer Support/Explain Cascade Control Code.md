FUNCTION_BLOCK FB_CascadePIDControl
VAR_INPUT
    PV1 : REAL;  // Measured pressure
    SP1 : REAL;  // Desired pressure setpoint
    PV2 : REAL;  // Measured flow rate
    dt  : REAL := 0.1; // Sample time (100 ms)
END_VAR

VAR_OUTPUT
    OP2 : REAL;  // Final control output to valve
END_VAR

VAR
    // Pressure loop PID parameters and variables
    e1, OP1 : REAL;
    e1_prev, i1, d1 : REAL;
    Kp1 : REAL := 1.2;
    Ki1 : REAL := 0.5;
    Kd1 : REAL := 0.1;

    // Flow loop PID parameters and variables
    SP2, e2 : REAL;
    e2_prev, i2, d2 : REAL;
    Kp2 : REAL := 2.5;
    Ki2 : REAL := 1.0;
    Kd2 : REAL := 0.3;
END_VAR

// --- Outer Loop (Pressure PID) ---
e1 := SP1 - PV1;
i1 := i1 + e1 * dt;
d1 := (e1 - e1_prev) / dt;
OP1 := (Kp1 * e1) + (Ki1 * i1) + (Kd1 * d1);

// Clamp OP1 to valid flow setpoint range (0–100%)
IF OP1 > 100.0 THEN
    OP1 := 100.0;
ELSIF OP1 < 0.0 THEN
    OP1 := 0.0;
END_IF

e1_prev := e1;
SP2 := OP1;  // Flow setpoint passed to inner loop

// --- Inner Loop (Flow PID) ---
e2 := SP2 - PV2;
i2 := i2 + e2 * dt;
d2 := (e2 - e2_prev) / dt;
OP2 := (Kp2 * e2) + (Ki2 * i2) + (Kd2 * d2);

// Clamp OP2 to valid valve control range (0–100%)
IF OP2 > 100.0 THEN
    OP2 := 100.0;
ELSIF OP2 < 0.0 THEN
    OP2 := 0.0;
END_IF

e2_prev := e2;

// Output OP2 to actuator
// SetValvePosition(OP2);  <-- Called externally by the main program
