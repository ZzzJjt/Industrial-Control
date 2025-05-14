PROGRAM PLC_PRG
TITLE 'Chlorine Dosing Control – PID Feedback'

(*
    Description:
    A PID-based control system for maintaining precise chlorine concentration
    in a water treatment process.

    Features:
    - Maintains target chlorine concentration (e.g., 3.0 ppm)
    - Uses real-time measurement and PID feedback control
    - Adjusts dosing output based on deviation from setpoint
    - Includes integral windup protection and output clamping
    - Modular design suitable for integration with SCADA or HMI systems

    Safety & Performance:
    - Prevents unsafe chlorine levels due to manual or open-loop methods
    - Stabilizes dosing under variable flow rates and disturbances
    - Easy to extend with diagnostics, alarms, or feedforward compensation
*)

// Constants and Tuning Parameters
CONST
    // Sampling time (assumed fixed control cycle)
    SAMPLE_TIME : REAL := 0.1;     // 100 ms = 0.1 seconds

    // Setpoint
    DOSING_SETPOINT : REAL := 3.0; // Target chlorine level in ppm

    // PID Gains
    KP : REAL := 2.0;   // Proportional gain
    KI : REAL := 0.5;   // Integral gain
    KD : REAL := 0.1;   // Derivative gain

    // Output Limits
    MIN_DOSE : REAL := 0.0;   // Minimum allowable dosing output
    MAX_DOSE : REAL := 10.0;  // Maximum allowable dosing output
END_CONST

VAR
    // Inputs
    Dosing_PV : REAL := 0.0;     // Measured chlorine level (ppm)

    // Outputs
    Dosing_Output : REAL := 0.0; // Output to dosing pump (e.g., % valve opening or pump speed)

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
    Error := DOSING_SETPOINT - Dosing_PV;

    // Update integral with anti-windup
    Integral := Integral + (Error * SAMPLE_TIME);
    
    // Optional: Clamp integral term to avoid windup
    IF Integral > MAX_DOSE THEN
        Integral := MAX_DOSE;
    ELSIF Integral < MIN_DOSE THEN
        Integral := MIN_DOSE;
    END_IF;

    // Compute derivative
    Derivative := (Error - Prev_Error) / SAMPLE_TIME;

    // Apply PID formula
    Dosing_Output := (KP * Error) + (KI * Integral) + (KD * Derivative);

    // Clamp output to safe range
    IF Dosing_Output > MAX_DOSE THEN
        Dosing_Output := MAX_DOSE;
    ELSIF Dosing_Output < MIN_DOSE THEN
        Dosing_Output := MIN_DOSE;
    END_IF;

    // Save current error for next derivative calculation
    Prev_Error := Error;
END_IF;

// Dosing_Output is now ready to be sent to the dosing pump controller
// e.g., analog output or communication link to chemical injection system

END_PROGRAM
