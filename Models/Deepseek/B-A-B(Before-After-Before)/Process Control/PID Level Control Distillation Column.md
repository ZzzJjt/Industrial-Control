PROGRAM PLC_PRG
TITLE 'Distillation Column Level Control – PID Feedback'

(*
    Description:
    A PID-based control system for maintaining precise liquid level in a distillation column.

    Features:
    - Maintains target liquid level (e.g., 60%) using real-time measurement
    - Adjusts inlet valve position via PID feedback
    - Includes anti-windup protection for integral term
    - Clamps output within safe valve operating range
    - Modular design suitable for SCADA integration or HMI tuning

    Safety & Performance:
    - Prevents flooding or tray drying due to poor level control
    - Stabilizes operation under variable feed or reflux conditions
    - Easy to extend with alarms, diagnostics, or feedforward compensation
*)

// Constants and Tuning Parameters
CONST
    // Sampling time (assumed fixed control cycle)
    SAMPLE_TIME : REAL := 0.1;     // 100 ms = 0.1 seconds

    // Setpoint
    LEVEL_SETPOINT : REAL := 60.0; // Target liquid level (%)

    // PID Gains
    KP : REAL := 1.5;   // Proportional gain
    KI : REAL := 0.4;   // Integral gain
    KD : REAL := 0.2;   // Derivative gain

    // Output Limits
    VALVE_MIN : REAL := 0.0;   // Minimum valve opening (%)
    VALVE_MAX : REAL := 100.0; // Maximum valve opening (%)
END_CONST

VAR
    // Inputs
    Level_PV : REAL := 0.0;     // Measured liquid level (%)

    // Outputs
    Valve_Position : REAL := 0.0; // Output to inlet valve actuator (%)

    // Internal variables
    Error : REAL := 0.0;
    Prev_Error : REAL := 0.0;
    Integral : REAL := 0.0;
    Derivative : REAL := 0.0;

    // Timer for consistent sampling interval
    SampleTimer : TON;
    TimerStarted : BOOL := FALSE;
END_VAR

// === MAIN LOGIC ===

// Start the timer once if not already running
IF NOT TimerStarted THEN
    SampleTimer(IN := TRUE, PT := T#100ms);
    TimerStarted := TRUE;
END_IF;

// When the timer completes one cycle
IF SampleTimer.Q THEN
    SampleTimer(IN := FALSE);  // Reset timer
    TimerStarted := FALSE;

    // Calculate error
    Error := LEVEL_SETPOINT - Level_PV;

    // Update integral with anti-windup
    Integral := Integral + (Error * SAMPLE_TIME);
    
    // Clamp integral term to avoid windup
    IF Integral > VALVE_MAX THEN
        Integral := VALVE_MAX;
    ELSIF Integral < VALVE_MIN THEN
        Integral := VALVE_MIN;
    END_IF;

    // Compute derivative
    Derivative := (Error - Prev_Error) / SAMPLE_TIME;

    // Apply PID formula
    Valve_Position := (KP * Error) + (KI * Integral) + (KD * Derivative);

    // Clamp output to safe range
    IF Valve_Position > VALVE_MAX THEN
        Valve_Position := VALVE_MAX;
    ELSIF Valve_Position < VALVE_MIN THEN
        Valve_Position := VALVE_MIN;
    END_IF;

    // Save current error for next derivative calculation
    Prev_Error := Error;
END_IF;

// Valve_Position is now ready to be sent to the actuator or VFD
// e.g., analog output to control valve or motorized inlet device

END_PROGRAM
