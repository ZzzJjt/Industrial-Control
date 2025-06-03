VAR
    // === Outer Loop (Temperature Control) ===
    Temp_SP : REAL := 85.0;        // Temperature setpoint (°C)
    Temp_PV : REAL;                // Measured outlet temperature (°C)
    Temp_Error : REAL;            // Error in temperature
    Temp_Output : REAL;           // Output of temperature controller (flow SP)

    // === Inner Loop (Flow Control) ===
    Flow_SP : REAL;               // Derived from Temp_Output
    Flow_PV : REAL;               // Measured flow rate (e.g., L/min)
    Flow_Error : REAL;            // Error in flow
    Flow_Output : REAL;           // Valve control signal (0.0–100.0%)

    // === Tuning Parameters ===
    Kp_Outer : REAL := 1.0;       // Outer loop proportional gain
    Kp_Inner : REAL := 2.0;       // Inner loop proportional gain

    // === Output Interface ===
    ControlValveOutput : REAL;    // Final output to actuator
END_VAR

// === Outer Loop PID (Proportional only for simplicity) ===
Temp_Error := Temp_SP - Temp_PV;
Temp_Output := Kp_Outer * Temp_Error;

// Clamp to safe flow range (e.g., 0–100 L/min)
IF Temp_Output < 0.0 THEN
    Flow_SP := 0.0;
ELSIF Temp_Output > 100.0 THEN
    Flow_SP := 100.0;
ELSE
    Flow_SP := Temp_Output;
END_IF

// === Inner Loop PID (Proportional only) ===
Flow_Error := Flow_SP - Flow_PV;
Flow_Output := Kp_Inner * Flow_Error;

// Clamp output to actuator limits (e.g., 0–100% valve open)
IF Flow_Output < 0.0 THEN
    ControlValveOutput := 0.0;
ELSIF Flow_Output > 100.0 THEN
    ControlValveOutput := 100.0;
ELSE
    ControlValveOutput := Flow_Output;
END_IF
