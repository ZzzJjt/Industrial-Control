PROGRAM CascadeControl
VAR
    // Primary (Outer) Loop Variables
    SP1 : REAL := 5.0; // Setpoint for pressure (bar)
    PV1 : REAL;        // Process Variable for pressure (measured pressure)
    Kp1 : REAL := 2.0; // Proportional gain for pressure loop
    Ki1 : REAL := 0.8; // Integral gain for pressure loop
    Kd1 : REAL := 0.3; // Derivative gain for pressure loop
    e1 : REAL;         // Error for pressure loop
    e1_sum : REAL := 0.0; // Integral term for pressure loop
    e1_prev : REAL := 0.0; // Previous error for pressure loop
    e1_diff : REAL;    // Derivative term for pressure loop
    OP1 : REAL;        // Output from pressure loop (used as setpoint for flow loop)

    // Secondary (Inner) Loop Variables
    SP2 : REAL;        // Setpoint for flow (calculated from OP1)
    PV2 : REAL;        // Process Variable for flow (measured flow rate)
    Kp2 : REAL := 1.5; // Proportional gain for flow loop
    Ki2 : REAL := 0.4; // Integral gain for flow loop
    Kd2 : REAL := 0.2; // Derivative gain for flow loop
    e2 : REAL;         // Error for flow loop
    e2_sum : REAL := 0.0; // Integral term for flow loop
    e2_prev : REAL := 0.0; // Previous error for flow loop
    e2_diff : REAL;    // Derivative term for flow loop
    OP2 : REAL;        // Output from flow loop (valve position)

    // Time variables
    dt : TIME := T#100ms; // Sampling time

    // Safety constraints
    Valve_Min : REAL := 0.0;
    Valve_Max : REAL := 100.0;

    // Timers for sampling
    Timer_Pressure : TON; // Timer for pressure loop
    Timer_Flow : TON;     // Timer for flow loop
END_VAR

// Initialize timers with a period of 100 ms
Timer_Pressure(PT := T#100ms);
Timer_Flow(PT := T#100ms);

// Main control loop
WHILE TRUE DO
    // Pressure Control Loop
    IF Timer_Pressure.Q THEN
        // Read measured pressure
        PV1 := ReadPressure();

        // Calculate error
        e1 := SP1 - PV1;

        // Update integral term
        e1_sum := e1_sum + e1 * dt;

        // Update derivative term
        e1_diff := (e1 - e1_prev) / dt;

        // Calculate PID output for pressure loop
        OP1 := (Kp1 * e1) + (Ki1 * e1_sum) + (Kd1 * e1_diff);

        // Clamp output to safe range
        IF OP1 > Valve_Max THEN
            OP1 := Valve_Max;
        ELSIF OP1 < Valve_Min THEN
            OP1 := Valve_Min;
        END_IF;

        // Store current error as previous error for next iteration
        e1_prev := e1;

        // Reset the timer
        Timer_Pressure(IN := FALSE);
    END_IF;

    // Flow Control Loop
    IF Timer_Flow.Q THEN
        // Read measured flow rate
        PV2 := ReadFlowRate();

        // Use pressure controller output as flow setpoint
        SP2 := OP1;

        // Calculate error
        e2 := SP2 - PV2;

        // Update integral term
        e2_sum := e2_sum + e2 * dt;

        // Update derivative term
        e2_diff := (e2 - e2_prev) / dt;

        // Calculate PID output for flow loop
        OP2 := (Kp2 * e2) + (Ki2 * e2_sum) + (Kd2 * e2_diff);

        // Clamp output to safe range
        IF OP2 > Valve_Max THEN
            OP2 := Valve_Max;
        ELSIF OP2 < Valve_Min THEN
            OP2 := Valve_Min;
        END_IF;

        // Store current error as previous error for next iteration
        e2_prev := e2;

        // Actuate valve position
        SetValvePosition(OP2);

        // Reset the timer
        Timer_Flow(IN := FALSE);
    END_IF;

    // Start the timers
    Timer_Pressure(IN := TRUE);
    Timer_Flow(IN := TRUE);

    // Delay for the next cycle
    SLEEP(dt);
END_WHILE;
