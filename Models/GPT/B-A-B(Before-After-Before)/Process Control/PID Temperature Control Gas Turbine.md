VAR
    // === Inputs ===
    Temp_PV : REAL;              // Measured gas turbine temperature (°C)
    Temp_SP : REAL := 950.0;     // Target setpoint temperature (°C)

    // === PID Tuning Parameters ===
    Kp : REAL := 3.0;            // Proportional gain
    Ki : REAL := 0.7;            // Integral gain
    Kd : REAL := 0.2;            // Derivative gain

    // === PID Internal Variables ===
    Error : REAL;
    Prev_Error : REAL := 0.0;
    Integral : REAL := 0.0;
    Derivative : REAL;

    // === Output to Inlet Valve ===
    Valve_Position : REAL;       // % open

    // === Valve Safety Limits ===
    Valve_Min : REAL := 0.0;
    Valve_Max : REAL := 100.0;

    // === Sampling Time ===
    SampleTime : REAL := 0.1;    // 100 ms
END_VAR

// === PID Calculation ===
Error := Temp_SP - Temp_PV;
Integral := Integral + Error * SampleTime;
Derivative := (Error - Prev_Error) / SampleTime;

Valve_Position := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

// === Clamp Output to Safe Range ===
IF Valve_Position > Valve_Max THEN
    Valve_Position := Valve_Max;
ELSIF Valve_Position < Valve_Min THEN
    Valve_Position := Valve_Min;
END_IF;

Prev_Error := Error;

// === Valve_Position can be used to drive an analog actuator ===
