PROGRAM HeatExchangerCascadeControl
VAR
    // === Outer Loop: Temperature Controller ===
    Temp_SP      : REAL := 85.0;      // Temperature Setpoint [°C]
    Temp_PV      : REAL;              // Process Variable: Actual Temperature [°C]
    Temp_Error   : REAL;              // Error = SP - PV
    Temp_Output  : REAL;              // Output: Flow Setpoint from Temp Controller

    // === Inner Loop: Flow Controller ===
    Flow_SP      : REAL;              // Setpoint from Outer Loop [e.g., L/min]
    Flow_PV      : REAL;              // Process Variable: Measured Flow [L/min]
    Flow_Error   : REAL;              // Error = SP - PV
    Flow_Output  : REAL;              // Output to Flow Control Valve

    // === Controller Gains ===
    Kp_Outer     : REAL := 1.0;       // Proportional Gain for Temp Loop
    Kp_Inner     : REAL := 2.0;       // Proportional Gain for Flow Loop

    // === Final Actuator Signal ===
    ValveCommand : REAL;              // Signal to actuate the control valve
END_VAR

// === Outer Loop Control ===
Temp_Error := Temp_SP - Temp_PV;              // Temperature error
Temp_Output := Kp_Outer * Temp_Error;         // Proportional output
Flow_SP := Temp_Output;                       // Flow setpoint for inner loop

// === Inner Loop Control ===
Flow_Error := Flow_SP - Flow_PV;              // Flow error
Flow_Output := Kp_Inner * Flow_Error;         // Proportional output
ValveCommand := Flow_Output;                  // Command to valve actuator
