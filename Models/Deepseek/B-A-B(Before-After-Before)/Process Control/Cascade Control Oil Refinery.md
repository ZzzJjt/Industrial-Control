PROGRAM PLC_PRG
TITLE 'Refinery Vessel Cascade Control – Pressure & Flow'

(*
    Description:
    A cascade control system for maintaining stable pressure in a refinery vessel.
    
    Features:
    - Outer loop controls vessel pressure (slow dynamics)
    - Inner loop adjusts oil inflow rate (fast dynamics)
    - Cascade structure improves disturbance rejection
    - Simple proportional control used for both loops
    - Clear variable naming and modular design
    
    Safety & Performance:
    - Prevents overshoot by separating fast/slow dynamics
    - Improves stability under load variations or inflow disturbances
    - Easy to extend with integral/derivative terms or limits
*)

// Constants and Tuning Parameters
CONST
    // Sampling time (assumed fixed control cycle)
    CONTROL_CYCLE : TIME := T#100ms;

    // Outer Loop (Pressure)
    PRESSURE_SETPOINT : REAL := 12.0;     // Target pressure in bar
    KP_OUTER        : REAL := 1.2;        // Proportional gain for outer loop

    // Inner Loop (Flow)
    KP_INNER        : REAL := 2.5;        // Proportional gain for inner loop

    // System Limits
    MIN_FLOW_OUTPUT : REAL := 0.0;        // Minimum valve opening (%)
    MAX_FLOW_OUTPUT : REAL := 100.0;      // Maximum valve opening (%)
END_CONST

VAR
    // Inputs
    Pressure_PV     : REAL := 0.0;        // Measured vessel pressure (bar)
    Flow_PV         : REAL := 0.0;        // Measured oil flow rate (m³/hr)

    // Outputs
    ValveOutput     : REAL := 0.0;        // Output to control valve (%)

    // Internal Variables
    Pressure_Error  : REAL := 0.0;
    Flow_SP         : REAL := 0.0;        // Setpoint from outer loop
    Flow_Error      : REAL := 0.0;
    Flow_Output     : REAL := 0.0;

    // Timer (for consistent update timing if needed)
    ControlTimer    : TON;
END_VAR

// === MAIN LOGIC ===

// Start the control timer (simulate fixed update interval)
ControlTimer(IN := TRUE, PT := CONTROL_CYCLE);

// Only perform control actions when timer is done
IF ControlTimer.Q THEN
    ControlTimer(IN := FALSE);  // Reset timer for next cycle

    // --- OUTER LOOP: Pressure Control ---
    Pressure_Error := PRESSURE_SETPOINT - Pressure_PV;
    Flow_SP := KP_OUTER * Pressure_Error;

    // Limit output to valid range for inner loop
    IF Flow_SP > MAX_FLOW_OUTPUT THEN
        Flow_SP := MAX_FLOW_OUTPUT;
    ELSIF Flow_SP < MIN_FLOW_OUTPUT THEN
        Flow_SP := MIN_FLOW_OUTPUT;
    END_IF;

    // --- INNER LOOP: Flow Control ---
    Flow_Error := Flow_SP - Flow_PV;
    Flow_Output := KP_INNER * Flow_Error;

    // Apply final limit before sending to actuator
    IF Flow_Output > MAX_FLOW_OUTPUT THEN
        ValveOutput := MAX_FLOW_OUTPUT;
    ELSIF Flow_Output < MIN_FLOW_OUTPUT THEN
        ValveOutput := MIN_FLOW_OUTPUT;
    ELSE
        ValveOutput := Flow_Output;
    END_IF;

END_IF;

END_PROGRAM
