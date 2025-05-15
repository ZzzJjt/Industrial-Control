PROGRAM CascadeControlVesselPressure
VAR_INPUT
    PV1 : REAL; // Measured vessel pressure
    PV2 : REAL; // Measured flow rate
END_VAR

VAR_OUTPUT
    OP2 : REAL; // Output to drive the valve (% open)
END_VAR

VAR
    // Primary Loop Variables
    SP1 : REAL := 10.0; // Desired setpoint for vessel pressure
    Kp1 : REAL := 2.0;
    Ki1 : REAL := 0.5;
    Kd1 : REAL := 0.1;
    e1 : REAL;
    e1_sum : REAL := 0.0;
    e1_prev : REAL := 0.0;
    OP1 : REAL;

    // Secondary Loop Variables
    SP2 : REAL;
    Kp2 : REAL := 3.0;
    Ki2 : REAL := 0.8;
    Kd2 : REAL := 0.2;
    e2 : REAL;
    e2_sum : REAL := 0.0;
    e2_prev : REAL := 0.0;

    // Sampling Time
    dt : TIME := T#100ms;

    // Output Clamping Limits
    Min_Output : REAL := 0.0;
    Max_Output : REAL := 100.0;
END_VAR

// PRIMARY LOOP: Pressure Control
e1 := SP1 - PV1;
e1_sum := e1_sum + e1 * TD_TO_S(dt);
e1_diff := (e1 - e1_prev) / TD_TO_S(dt);
OP1 := Kp1 * e1 + Ki1 * e1_sum + Kd1 * e1_diff;

// Clamp OP1 to [Min_Output, Max_Output]
IF OP1 > Max_Output THEN
    OP1 := Max_Output;
ELSIF OP1 < Min_Output THEN
    OP1 := Min_Output;
END_IF;

// Assign OP1 as the setpoint for the secondary loop
SP2 := OP1;

// SECONDARY LOOP: Flow Control
e2 := SP2 - PV2;
e2_sum := e2_sum + e2 * TD_TO_S(dt);
e2_diff := (e2 - e2_prev) / TD_TO_S(dt);
OP2 := Kp2 * e2 + Ki2 * e2_sum + Kd2 * e2_diff;

// Clamp OP2 to [Min_Output, Max_Output]
IF OP2 > Max_Output THEN
    OP2 := Max_Output;
ELSIF OP2 < Min_Output THEN
    OP2 := Min_Output;
END_IF;

// Update previous errors for next iteration
e1_prev := e1;
e2_prev := e2;

// Inline comments explaining the logic:
// The program implements a cascade control scheme for regulating vessel pressure using two PID loops.
// The primary loop regulates the vessel pressure (PV1) by comparing it to the setpoint (SP1).
// The PID controller calculates the error (e1), integral (e1_sum), and derivative (e1_diff) terms,
// generating OP1 as the setpoint for the secondary loop.
// The secondary loop regulates the flow rate (PV2) by comparing it to the setpoint (SP2 = OP1).
// Another PID controller calculates the error (e2), integral (e2_sum), and derivative (e2_diff) terms,
// generating OP2 as the valve position.
// Both loops include anti-windup output clamping (0.0–100.0) to prevent saturation of valve commands.
// A fixed sample time (dt := t#100ms) is used for integral and derivative calculations.



