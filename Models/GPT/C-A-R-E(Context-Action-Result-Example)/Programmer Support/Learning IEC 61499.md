FUNCTION_BLOCK FB_TemperatureControl
VAR_INPUT
    Temp_IN : REAL;           // Measured temperature
    SetPoint : REAL := 75.0;  // Desired temperature
    Kp : REAL := 1.0;
    Ki : REAL := 0.2;
    Kd : REAL := 0.1;
END_VAR

VAR_OUTPUT
    Output : REAL;            // Control signal to actuator
    Error : REAL;
END_VAR

VAR
    eSum : REAL := 0.0;        // Integral term
    prevError : REAL := 0.0;   // Derivative term
    dt : REAL := 0.1;          // Time step in seconds
END_VAR

EVENT_INPUT REQ
EVENT_OUTPUT CNF

ALGORITHM PID
BEGIN
    Error := SetPoint - Temp_IN;
    eSum := eSum + Error * dt;
    Output := Kp * Error + Ki * eSum + Kd * (Error - prevError) / dt;
    prevError := Error;
END_ALGORITHM

EVENT REQ => PID => CNF;
