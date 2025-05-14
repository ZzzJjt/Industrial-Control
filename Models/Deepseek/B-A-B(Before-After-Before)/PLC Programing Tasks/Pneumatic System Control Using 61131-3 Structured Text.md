PROGRAM PLC_PRG
TITLE 'Pneumatic System Controller – Flow & Pressure Regulation'

(*
    Description:
    Controls a pneumatic system by regulating air flow and pressure.
    
    Features:
    - Runs on a fixed 100 ms control loop
    - Maintains flow at 50 SLPM using a valve output
    - Monitors pressure between 5.5 and 6.0 bar
    - Activates safety mechanisms if limits are exceeded
    - Flags errors for diagnostics or HMI integration
    
    Safety:
    - Detects and responds to flow deviation (> ±5 SLPM)
    - Shuts down or vents pressure if out of range
*)

// Configuration Constants
CONST
    CONTROL_LOOP_TIME : TIME := T#100ms;
    FLOW_SETPOINT     : REAL := 50.0;   // Target flow in SLPM
    FLOW_TOLERANCE    : REAL := 5.0;    // Acceptable deviation from setpoint
    MIN_PRESSURE      : REAL := 5.5;    // Minimum safe pressure (bar)
    MAX_PRESSURE      : REAL := 6.0;    // Maximum safe pressure (bar)
END_CONST

VAR
    // Inputs
    FlowInput        : REAL := 0.0;    // Measured flow rate (SLPM)
    PressureInput    : REAL := 0.0;    // Measured pressure (bar)

    // Outputs
    FlowValveOutput  : BOOL := FALSE;  // Controls inlet valve to adjust flow
    ReliefValve      : BOOL := FALSE;  // Activates pressure relief mechanism

    // Internal Flags
    FlowError        : BOOL := FALSE;  // TRUE if flow deviates beyond tolerance
    PressureError    : BOOL := FALSE;  // TRUE if pressure outside safe range
    SystemActive     : BOOL := TRUE;   // Enables/disables the system

    // Timer for consistent 100 ms control loop
    ControlTimer     : TON;
END_VAR

// === MAIN LOGIC ===

// Start the 100 ms control timer
ControlTimer(IN := TRUE, PT := CONTROL_LOOP_TIME);

// Only perform control actions when timer is done
IF ControlTimer.Q THEN
    ControlTimer(IN := FALSE);  // Reset timer for next cycle

    // --- FLOW CONTROL ---
    IF FlowInput < (FLOW_SETPOINT - FLOW_TOLERANCE) THEN
        FlowValveOutput := TRUE;  // Open valve to increase flow
        FlowError := TRUE;

    ELSIF FlowInput > (FLOW_SETPOINT + FLOW_TOLERANCE) THEN
        FlowValveOutput := FALSE; // Close valve to reduce flow
        FlowError := TRUE;

    ELSE
        FlowValveOutput := TRUE;  // Maintain minimal open state to sustain flow
        FlowError := FALSE;
    END_IF;

    // --- PRESSURE MONITORING ---
    IF (PressureInput < MIN_PRESSURE) OR (PressureInput > MAX_PRESSURE) THEN
        PressureError := TRUE;
        ReliefValve := TRUE;  // Activate pressure relief
    ELSE
        PressureError := FALSE;
        ReliefValve := FALSE;
    END_IF;

    // --- SYSTEM SAFETY INTERLOCK ---
    IF NOT SystemActive THEN
        FlowValveOutput := FALSE;
        ReliefValve := TRUE;  // Optionally vent residual pressure
    END_IF;

END_IF;

END_PROGRAM
