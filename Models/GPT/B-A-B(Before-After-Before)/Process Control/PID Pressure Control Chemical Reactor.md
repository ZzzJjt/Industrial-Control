VAR
    // === Inputs ===
    Pressure_PV : REAL;               // Measured reactor pressure (bar)
    Pressure_SP : REAL := 5.0;        // Pressure setpoint (target value)

    // === PID Parameters ===
    Kp : REAL := 2.0;                 // Proportional gain
    Ki : REAL := 0.8;                 // Integral gain
    Kd : REAL := 0.3;                 // Derivative gain

    // === PID Internal Variables ===
    Error : REAL;                     // Current error
    Prev_Error : REAL := 0.0;         // Previous cycle's error
    Integral : REAL := 0.0;           // Accumulated integral
    Derivative : REAL;                // Rate of change

    // === Output ===
    Valve_Output : REAL;              // Control valve position (%)

    // === Output Limits ===
    Valve_Min : REAL := 0.0;          // Minimum valve opening
    Valve_Max : REAL := 100.0;        // Maximum valve opening

    // === Sampling Time ===
    SampleTime : REAL := 0.1;         // In seconds (100 ms)
END_VAR

// === PID Logic ===
Error := Pressure_SP - Pressure_PV;
Integral := Integral + Error * SampleTime;
Derivative := (Error - Prev_Error) / SampleTime;

Valve_Output := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

// === Clamp Valve Output ===
IF Valve_Output > Valve_Max THEN
    Valve_Output := Valve_Max;
ELSIF Valve_Output < Valve_Min THEN
    Valve_Output := Valve_Min;
END_IF;

Prev_Error := Error;

// === Valve_Output can be connected to an analog output to control the valve ===
