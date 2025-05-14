PROGRAM PLC_PRG
TITLE 'Chemical Reactor Pressure Control – PID Feedback'

(*
    Description:
    A PID-based control system for maintaining precise pressure in a chemical reactor.

    Features:
    - Maintains target reactor pressure using real-time measurement
    - Adjusts pressure control valve via PID feedback
    - Includes anti-windup protection for integral term
    - Clamps output within safe valve operating range
    - Modular design suitable for SCADA integration or HMI tuning

    Safety & Performance:
    - Prevents over-pressurization risks or incomplete reactions
    - Stabilizes operation under dynamic process conditions
    - Easy to extend with alarms, diagnostics, or feedforward compensation
*)

// Constants and Tuning Parameters
CONST
    // Sampling time (assumed fixed control cycle)
    SAMPLE_TIME : REAL := 0.1;     // 100 ms = 0.1 seconds

    // Setpoint
    PRESSURE_SETPOINT : REAL := 5.0; // Target reactor pressure (e.g., bar)

    // PID Gains
    KP : REAL := 2.0;   // Proportional gain
    KI : REAL := 0.8;   // Integral gain
    KD : REAL := 0.3;   // Derivative gain

    // Output Limits
    VALVE_MIN : REAL := 0.0;   // Minimum valve opening (%)
    VALVE_MAX : REAL := 100.0; // Maximum valve opening (%)
END_CONST

VAR
    // Inputs
    Pressure_PV : REAL := 0.0;     // Measured reactor pressure (bar)

    // Outputs
    Valve_Output : REAL := 0.0;    // Output to pressure control valve actuator (%)

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
    Error := PRESSURE_SETPOINT - Pressure_PV;

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
    Valve_Output := (KP * Error) + (KI * Integral) + (KD * Derivative);

    // Clamp output to safe range
    IF Valve_Output > VALVE_MAX THEN
        Valve_Output := VALVE_MAX;
    ELSIF Valve_Output < VALVE_MIN THEN
        Valve_Output := VALVE_MIN;
    END_IF;

    // Save current error for next derivative calculation
    Prev_Error := Error;
END_IF;

// Valve_Output is now ready to be sent to the valve actuator
// e.g., analog output to control valve or motorized exhaust device

END_PROGRAM
