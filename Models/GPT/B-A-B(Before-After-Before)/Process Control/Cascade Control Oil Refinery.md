VAR
    // === Outer Loop: Pressure Control ===
    Pressure_SP : REAL := 12.0;       // Pressure setpoint (bar)
    Pressure_PV : REAL;               // Measured vessel pressure
    Pressure_Error : REAL;           // Pressure control error
    Flow_SP : REAL;                  // Setpoint for flow, driven by pressure loop

    // === Inner Loop: Flow Control ===
    Flow_PV : REAL;                  // Measured oil flow rate
    Flow_Error : REAL;               // Flow control error
    Flow_Output : REAL;              // Final actuator signal (e.g., valve position %)

    // === Controller Gains ===
    Kp_Outer : REAL := 1.2;          // Pressure controller gain
    Kp_Inner : REAL := 2.5;          // Flow controller gain

    // === Actuator Output ===
    OilInletValve : REAL;           // Output to control valve (0–100%)
END_VAR

// === Outer Loop: Pressure PID (Proportional Only) ===
Pressure_Error := Pressure_SP - Pressure_PV;
Flow_SP := Kp_Outer * Pressure_Error;

// Clamp flow setpoint to safe range (e.g., 0–100 L/min)
IF Flow_SP < 0.0 THEN
    Flow_SP := 0.0;
ELSIF Flow_SP > 100.0 THEN
    Flow_SP := 100.0;
END_IF

// === Inner Loop: Flow PID (Proportional Only) ===
Flow_Error := Flow_SP - Flow_PV;
Flow_Output := Kp_Inner * Flow_Error;

// Clamp output to actuator range
IF Flow_Output < 0.0 THEN
    OilInletValve := 0.0;
ELSIF Flow_Output > 100.0 THEN
    OilInletValve := 100.0;
ELSE
    OilInletValve := Flow_Output;
END_IF
