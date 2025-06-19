VAR
    // === Outer loop: Pressure Control ===
    SP1 : REAL := 12.0;            // Pressure setpoint (e.g., bar)
    PV1 : REAL;                    // Measured vessel pressure
    e1 : REAL;                     // Pressure error
    OP1 : REAL;                    // Output of pressure controller (setpoint for flow)

    Kp1 : REAL := 1.5;             // Pressure loop P gain
    Ki1 : REAL := 0.4;             // Pressure loop I gain
    Kd1 : REAL := 0.2;             // Pressure loop D gain

    I1 : REAL := 0.0;              // Pressure loop integral
    D1 : REAL;                     // Pressure loop derivative
    e1_prev : REAL := 0.0;

    // === Inner loop: Flow Control ===
    SP2 : REAL;                    // Flow setpoint (from OP1)
    PV2 : REAL;                    // Measured flow rate
    e2 : REAL;                     // Flow error
    OP2 : REAL;                    // Output of flow controller (valve position)

    Kp2 : REAL := 2.0;             // Flow loop P gain
    Ki2 : REAL := 0.6;             // Flow loop I gain
    Kd2 : REAL := 0.1;             // Flow loop D gain

    I2 : REAL := 0.0;              // Flow loop integral
    D2 : REAL;                     // Flow loop derivative
    e2_prev : REAL := 0.0;

    // === Common settings ===
    dt : REAL := 0.1;              // Sample time in seconds
    OP_MIN : REAL := 0.0;
    OP_MAX : REAL := 100.0;
END_VAR

// === Read process values ===
PV1 := ReadPressure();             // Replace with actual pressure sensor reading
PV2 := ReadFlowRate();            // Replace with actual flow sensor reading

// === Outer loop PID: Pressure ===
e1 := SP1 - PV1;
I1 := I1 + e1 * dt;
D1 := (e1 - e1_prev) / dt;
OP1 := (Kp1 * e1) + (Ki1 * I1) + (Kd1 * D1);
e1_prev := e1;

// Clamp outer loop output
IF OP1 > OP_MAX THEN
    OP1 := OP_MAX;
ELSIF OP1 < OP_MIN THEN
    OP1 := OP_MIN;
END_IF;

SP2 := OP1; // Inner loop setpoint from outer loop output

// === Inner loop PID: Flow ===
e2 := SP2 - PV2;
I2 := I2 + e2 * dt;
D2 := (e2 - e2_prev) / dt;
OP2 := (Kp2 * e2) + (Ki2 * I2) + (Kd2 * D2);
e2_prev := e2;

// Clamp inner loop output
IF OP2 > OP_MAX THEN
    OP2 := OP_MAX;
ELSIF OP2 < OP_MIN THEN
    OP2 := OP_MIN;
END_IF;

// === Actuator Output ===
SetValvePosition(OP2);            // Replace with actual valve control function
