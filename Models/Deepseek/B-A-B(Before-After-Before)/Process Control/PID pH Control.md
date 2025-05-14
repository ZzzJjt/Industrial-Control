PROGRAM PLC_PRG
TITLE 'Industrial pH Control – PID Feedback'

(*
    Description:
    A PID-based control system for maintaining precise pH levels in an industrial process.

    Features:
    - Maintains target pH using real-time measurement
    - Adjusts acid/base dosing pump via PID feedback
    - Includes anti-windup protection for integral term
    - Clamps output within safe dosing range
    - Modular design suitable for SCADA integration or HMI tuning

    Safety & Performance:
    - Prevents instability, corrosion, or product quality issues due to pH drift
    - Stabilizes operation under variable feedstock or load conditions
    - Easy to extend with alarms, diagnostics, or dual-dosing logic
*)

// Constants and Tuning Parameters
CONST
    // Sampling time (assumed fixed control cycle)
    SAMPLE_TIME : REAL := 0.1;     // 100 ms = 0.1 seconds

    // Setpoint
    PH_SETPOINT : REAL := 7.0;      // Target pH level

    // PID Gains
    KP : REAL := 2.5;   // Proportional gain
    KI : REAL := 0.6;   // Integral gain
    KD : REAL := 0.3;   // Derivative gain

    // Output Limits
    DOSING_MIN : REAL := 0.0;   // Minimum dosing output (%)
    DOSING_MAX : REAL := 100.0; // Maximum dosing output (%)
END_CONST

VAR
    // Inputs
    pH_PV : REAL := 0.0;     // Measured pH value (e.g., from sensor)

    // Outputs
    Dosing_Output : REAL := 0.0; // Output to dosing pump or valve (%)

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
    Error := PH_SETPOINT - pH_PV;

    // Update integral with anti-windup
    Integral := Integral + (Error * SAMPLE_TIME);
    
    // Clamp integral term to avoid windup
    IF Integral > DOSING_MAX THEN
        Integral := DOSING_MAX;
    ELSIF Integral < DOSING_MIN THEN
        Integral := DOSING_MIN;
    END_IF;

    // Compute derivative
    Derivative := (Error - Prev_Error) / SAMPLE_TIME;

    // Apply PID formula
    Dosing_Output := (KP * Error) + (KI * Integral) + (KD * Derivative);

    // Clamp output to safe range
    IF Dosing_Output > DOSING_MAX THEN
        Dosing_Output := DOSING_MAX;
    ELSIF Dosing_Output < DOSING_MIN THEN
        Dosing_Output := DOSING_MIN;
    END_IF;

    // Save current error for next derivative calculation
    Prev_Error := Error;
END_IF;

// Dosing_Output is now ready to be sent to the dosing pump or valve
// e.g., analog output to control acid/base addition

END_PROGRAM
