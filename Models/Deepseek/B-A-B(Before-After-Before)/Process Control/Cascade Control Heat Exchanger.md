PROGRAM PLC_PRG
TITLE 'Heat Exchanger Cascade Control – Temperature & Flow'

(*
    Description:
    A cascade control system for a heat exchanger.
    
    Features:
    - Outer loop controls process temperature (e.g., product outlet)
    - Inner loop adjusts flow rate of heating medium
    - Cascade structure improves disturbance rejection
    - Simple proportional control used for both loops
    - Clear variable naming and modular design
    
    Safety & Performance:
    - Prevents overshoot by separating fast/slow dynamics
    - Improves stability under load variations
    - Easy to extend with integral/derivative terms or limits
*)

// Constants and Tuning Parameters
CONST
    // Sampling time (assumed fixed control cycle)
    CONTROL_CYCLE : TIME := T#100ms;

    // Outer Loop (Temperature)
    TEMP_SETPOINT : REAL := 85.0;     // Target outlet temperature in °C
    KP_OUTER      : REAL := 2.0;      // Proportional gain for outer loop

    // Inner Loop (Flow)
    KP_INNER      : REAL := 1.5;      // Proportional gain for inner loop

    // System Limits
    MIN_FLOW_OUTPUT : REAL := 0.0;
    MAX_FLOW_OUTPUT : REAL := 100.0;  // Max valve opening %
END_CONST

VAR
    // Inputs
    Temp_PV       : REAL := 0.0;      // Measured outlet temperature (°C)
    Flow_PV       : REAL := 0.0;      // Measured flow rate of heating medium

    // Outputs
    ValveOutput   : REAL := 0.0;      // Output to control valve (%)

    // Internal Variables
    Temp_Error    : REAL := 0.0;
    Flow_SP       : REAL := 0.0;      // Setpoint from outer loop
    Flow_Error    : REAL := 0.0;
    Flow_Output   : REAL := 0.0;

    // Timer (for consistent update timing if needed)
    ControlTimer  : TON;
END_VAR

// === MAIN LOGIC ===

// Start the control timer (simulate fixed update interval)
ControlTimer(IN := TRUE, PT := CONTROL_CYCLE);

// Only perform control actions when timer is done
IF ControlTimer.Q THEN
    ControlTimer(IN := FALSE);  // Reset timer for next cycle

    // --- OUTER LOOP: Temperature Control ---
    Temp_Error := TEMP_SETPOINT - Temp_PV;
    Flow_SP := KP_OUTER * Temp_Error;

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
