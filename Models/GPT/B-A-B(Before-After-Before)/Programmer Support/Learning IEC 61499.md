FUNCTION_BLOCK FB_TemperatureControl
VAR_INPUT
    Temp_SP : REAL;         // Desired temperature setpoint
    Temp_PV : REAL;         // Measured temperature
    Kp      : REAL := 2.0;
    Ki      : REAL := 0.5;
    Kd      : REAL := 0.1;
    dt      : REAL := 0.1;  // Time step in seconds
END_VAR

VAR_OUTPUT
    Heater_Output : REAL;   // Output to heater actuator
END_VAR

VAR
    e        : REAL;        // Current error
    e_prev   : REAL := 0.0; // Previous error
    integral : REAL := 0.0;
    derivative : REAL;
END_VAR

EVENT_INPUT REQ
EVENT_OUTPUT CNF

ALGORITHM PIDControl
BEGIN
    e := Temp_SP - Temp_PV;
    integral := integral + e * dt;
    derivative := (e - e_prev) / dt;

    Heater_Output := (Kp * e) + (Ki * integral) + (Kd * derivative);

    // Clamp output to range [0..100]
    IF Heater_Output > 100.0 THEN
        Heater_Output := 100.0;
    ELSIF Heater_Output < 0.0 THEN
        Heater_Output := 0.0;
    END_IF;

    e_prev := e;
END_ALGORITHM

EC_STATE Start : INITIAL;
EC_STATE Running;
EC_TRANSITION Start -> Running ON REQ;
EC_TRANSITION Running -> Start ON CNF;

EC_ACTION Running := PIDControl;
