PROGRAM PLC_PRG
TITLE 'Reactant Ratio Control – 2:1 Mixing'

(*
    Description:
    This program maintains a 2:1 mixing ratio between two reactants (A:B)
    by dynamically adjusting the setpoint for Reactant B based on the measured flow of Reactant A.

    Features:
    - Calculates real-time actual ratio from flow measurements
    - Computes required flow rate for Reactant B to maintain target ratio
    - Detects deviations beyond acceptable tolerance
    - Provides output for flow controller or pump control
    - Optional alarm if ratio deviation exceeds tolerance

    Safety & Performance:
    - Prevents off-spec product due to incorrect mixing ratios
    - Stabilizes operation under variable flow conditions
    - Modular design suitable for SCADA integration or alarms
*)

// Constants
CONST
    // Target mixing ratio (A : B = 2 : 1)
    RATIO_SETPOINT : REAL := 2.0;

    // Acceptable deviation from target ratio before triggering alarm
    TOLERANCE : REAL := 0.05; // ±5% deviation

    // Sample time (assumed fixed cycle)
    SAMPLE_TIME : REAL := 0.1; // 100 ms
END_CONST

VAR
    // Inputs
    Flow_A_PV : REAL := 0.0; // Measured flow rate of Reactant A (e.g., L/min)
    Flow_B_PV : REAL := 0.0; // Measured flow rate of Reactant B (e.g., L/min)

    // Outputs
    Flow_B_SP : REAL := 0.0; // Calculated flow setpoint for Reactant B
    Ratio_Alarm : BOOL := FALSE; // Flag if ratio is out of tolerance

    // Internal variables
    Actual_Ratio : REAL := 0.0;
    Error : REAL := 0.0;
    TimerStarted : BOOL := FALSE;

    // Timer for consistent sampling interval
    SampleTimer : TON;
END_VAR

// === MAIN LOGIC ===

// Start timer once per scan if not already running
IF NOT TimerStarted THEN
    SampleTimer(IN := TRUE, PT := T#100ms);
    TimerStarted := TRUE;
END_IF;

// When the timer completes one cycle
IF SampleTimer.Q THEN
    SampleTimer(IN := FALSE);  // Reset timer
    TimerStarted := FALSE;

    // Calculate actual ratio only if Flow_B_PV is non-zero to avoid division by zero
    IF Flow_B_PV > 0.0 THEN
        Actual_Ratio := Flow_A_PV / Flow_B_PV;
    ELSE
        Actual_Ratio := 0.0;
    END_IF;

    // Compute required Flow_B to maintain 2:1 ratio
    IF RATIO_SETPOINT <> 0.0 THEN
        Flow_B_SP := Flow_A_PV / RATIO_SETPOINT;
    ELSE
        Flow_B_SP := 0.0;
    END_IF;

    // Calculate deviation from desired ratio
    Error := Actual_Ratio - RATIO_SETPOINT;

    // Check if error is outside of acceptable tolerance
    IF ABS(Error) > TOLERANCE THEN
        Ratio_Alarm := TRUE;
        // Optional: Trigger corrective action or send alarm signal
    ELSE
        Ratio_Alarm := FALSE;
    END_IF;

END_IF;

// Output Flow_B_SP can be sent to:
// - Analog output to control pump speed or valve position
// - PID loop controlling Reactant B flow
// - HMI for operator monitoring

// End of Program
END_PROGRAM
