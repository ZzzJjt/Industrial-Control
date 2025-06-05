PROGRAM CascadePressureFlowControl
VAR
    // --- Sample Time ---
    dt : REAL := 0.1; // Sample time in seconds (100 ms)

    // --- Primary Loop (Pressure Control) ---
    SP1      : REAL := 5.0;     // Pressure setpoint (bar)
    PV1      : REAL;            // Measured vessel pressure (bar)
    e1       : REAL;            // Pressure error
    e1_prev  : REAL := 0.0;     // Previous pressure error
    e1_sum   : REAL := 0.0;     // Integral accumulator
    e1_diff  : REAL;            // Derivative term
    OP1      : REAL;            // Output of primary controller (used as SP2)

    // Tuning parameters for pressure loop
    Kp1 : REAL := 3.0;
    Ki1 : REAL := 0.8;
    Kd1 : REAL := 0.5;

    // --- Secondary Loop (Flow Control) ---
    SP2      : REAL;            // Flow setpoint (inherited from OP1)
    PV2      : REAL;            // Measured flow rate (e.g., L/min)
    e2       : REAL;            // Flow error
    e2_prev  : REAL := 0.0;     // Previous flow error
    e2_sum   : REAL := 0.0;     // Integral accumulator
    e2_diff  : REAL;            // Derivative term
    OP2      : REAL;            // Final valve control output

    // Tuning parameters for flow loop
    Kp2 : REAL := 2.5;
    Ki2 : REAL := 1.2;
    Kd2 : REAL := 0.3;

    // --- Output Constraints ---
    OUT_MIN : REAL := 0.0;
    OUT_MAX : REAL := 100.0;
END_VAR

// --- PRIMARY LOOP: Pressure PID Controller ---
e1 := SP1 - PV1;
e1_sum := e1_sum + e1 * dt;
e1_diff := (e1 - e1_prev) / dt;

OP1 := (Kp1 * e1) + (Ki1 * e1_sum) + (Kd1 * e1_diff);

// Clamp OP1 (SP2) to safety range
IF OP1 > OUT_MAX THEN
    OP1 := OUT_MAX;
ELSIF OP1 < OUT_MIN THEN
    OP1 := OUT_MIN;
END_IF;

e1_prev := e1;

// --- SECONDARY LOOP: Flow PID Controller ---
SP2 := OP1;
e2 := SP2 - PV2;
e2_sum := e2_sum + e2 * dt;
e2_diff := (e2 - e2_prev) / dt;

OP2 := (Kp2 * e2) + (Ki2 * e2_sum) + (Kd2 * e2_diff);

// Clamp OP2 (valve command) to safety range
IF OP2 > OUT_MAX THEN
    OP2 := OUT_MAX;
ELSIF OP2 < OUT_MIN THEN
    OP2 := OUT_MIN;
END_IF;

e2_prev := e2;

// --- Apply Output to Actuator ---
SetValvePosition(OP2); // External actuator function
