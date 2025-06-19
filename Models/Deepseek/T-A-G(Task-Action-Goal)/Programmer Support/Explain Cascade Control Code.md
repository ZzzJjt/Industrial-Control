PROGRAM CascadeControl
VAR_INPUT
    ReadPressure : FUNCTION () : REAL; // Function to read current pressure (PV1)
    ReadFlowRate : FUNCTION () : REAL; // Function to read current flow rate (PV2)
END_VAR

VAR_OUTPUT
    SetValvePosition : FUNCTION (OP2 : REAL) : VOID; // Function to set valve position
END_VAR

VAR
    // Primary Loop - Pressure Control
    SP1 : REAL := 100.0; // Desired pressure setpoint
    PV1 : REAL;          // Measured pressure (Process Variable 1)
    OP1 : REAL;          // Output from primary loop (Flow Setpoint for secondary loop)
    Kp1 : REAL := 1.0;   // Proportional gain for primary loop
    Ki1 : REAL := 0.5;   // Integral gain for primary loop
    Kd1 : REAL := 0.1;   // Derivative gain for primary loop
    Error1 : REAL;
    Prev_Error1 : REAL := 0.0;
    Integral1 : REAL := 0.0;
    Derivative1 : REAL;

    // Secondary Loop - Flow Control
    SP2 : REAL;           // Desired flow setpoint (Output from primary loop)
    PV2 : REAL;           // Measured flow rate (Process Variable 2)
    OP2 : REAL;           // Output from secondary loop (Valve Position)
    Kp2 : REAL := 2.0;    // Proportional gain for secondary loop
    Ki2 : REAL := 1.0;    // Integral gain for secondary loop
    Kd2 : REAL := 0.2;    // Derivative gain for secondary loop
    Error2 : REAL;
    Prev_Error2 : REAL := 0.0;
    Integral2 : REAL := 0.0;
    Derivative2 : REAL;

    // Timing variables
    LastTime : TIME;      // Variable to store the last execution time
    CycleTime : TIME := T#100ms; // Control cycle time

    // Safety limits
    Valve_Min : REAL := 0.0;
    Valve_Max : REAL := 100.0;
END_VAR

// Check if the control cycle time has elapsed
IF TONR(T:=CycleTime, ET=>LastTime).Q THEN
    LastTime := TONR(T:=CycleTime, ET=>LastTime).ET;

    // Primary Loop - Pressure Control
    PV1 := ReadPressure(); // Read the current pressure
    Error1 := SP1 - PV1;   // Calculate the error

    // Accumulate the integral term
    Integral1 := Integral1 + Error1 * T_TO_S(CycleTime);

    // Compute the derivative term
    Derivative1 := (Error1 - Prev_Error1) / T_TO_S(CycleTime);

    // Calculate the PID output for primary loop
    OP1 := (Kp1 * Error1) + (Ki1 * Integral1) + (Kd1 * Derivative1);

    // Clamp the output within safe operational limits
    IF OP1 > Valve_Max THEN
        OP1 := Valve_Max;
    ELSIF OP1 < Valve_Min THEN
        OP1 := Valve_Min;
    END_IF;

    // Update previous error for the next iteration
    Prev_Error1 := Error1;

    // Assign OP1 as the setpoint for the secondary loop
    SP2 := OP1;

    // Secondary Loop - Flow Control
    PV2 := ReadFlowRate(); // Read the current flow rate
    Error2 := SP2 - PV2;   // Calculate the error

    // Accumulate the integral term
    Integral2 := Integral2 + Error2 * T_TO_S(CycleTime);

    // Compute the derivative term
    Derivative2 := (Error2 - Prev_Error2) / T_TO_S(CycleTime);

    // Calculate the PID output for secondary loop
    OP2 := (Kp2 * Error2) + (Ki2 * Integral2) + (Kd2 * Derivative2);

    // Clamp the output within safe operational limits
    IF OP2 > Valve_Max THEN
        OP2 := Valve_Max;
    ELSIF OP2 < Valve_Min THEN
        OP2 := Valve_Min;
    END_IF;

    // Set the valve position based on the output from the secondary loop
    SetValvePosition(OP2);

    // Update previous error for the next iteration
    Prev_Error2 := Error2;
END_IF;

// Additional comments for clarity
// - Primary Loop (Pressure Control):
//   - Reads the current pressure (PV1).
//   - Compares it to the desired pressure setpoint (SP1).
//   - Calculates the error and applies PID control using gains (Kp1, Ki1, Kd1).
//   - Outputs OP1, which becomes the flow setpoint (SP2) for the inner loop.
//   - Clamps OP1 between 0.0 and 100.0 to ensure safe operation.
//
// - Secondary Loop (Flow Control):
//   - Reads the current flow rate (PV2).
//   - Uses OP1 as the setpoint (SP2).
//   - Calculates the error and applies a second PID controller using gains (Kp2, Ki2, Kd2).
//   - Sets the valve position via SetValvePosition(OP2).
//   - Clamps OP2 between 0.0 and 100.0 to ensure safe operation.
//
// - Timing:
//   - Uses a fixed sample time (dt := t#100ms) for all integral and derivative calculations.



