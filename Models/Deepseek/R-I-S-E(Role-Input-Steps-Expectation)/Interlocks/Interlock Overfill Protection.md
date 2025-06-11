FUNCTION_BLOCK FB_OverfillProtection
VAR_INPUT
    rLevelSensor: REAL;        // Measured liquid level (% or mm)
    bValveStuckOpen: BOOL;     // Valve feedback indicates it's stuck open
    bValveStuckClosed: BOOL;   // Valve feedback indicates it's stuck closed
    bSensorFault: BOOL;        // Level sensor failure or out-of-range signal
    bManualReset: BOOL;        // Manual reset button pressed by operator
END_VAR

VAR_OUTPUT
    bInletValve_Open: BOOL;    // Inlet valve command – TRUE = Open, FALSE = Closed
    bHighLevelAlarm: BOOL;     // High-level alarm to SCADA or HMI
    bSystemLockedOut: BOOL;    // System locked out until manual reset
END_VAR

VAR
    // Configuration Constants
    HIGH_SETPOINT AT %I0.0: REAL := 90.0;       // High-level trip point (e.g., 90% full)
    RESET_THRESHOLD AT %I4.0: REAL := 80.0;     // Reset level before re-opening valve

    // Internal Flags
    bHighLevelDetected: BOOL := FALSE;
    bValveFaultDetected: BOOL := FALSE;
    bLockoutTriggered: BOOL := FALSE;
END_VAR

// Initialize outputs at start of scan
bInletValve_Open := FALSE;
bHighLevelAlarm := FALSE;
bSystemLockedOut := FALSE;

// Check for sensor faults first – highest priority
IF bSensorFault THEN
    bLockoutTriggered := TRUE;
    bHighLevelAlarm := TRUE;
    bSystemLockedOut := TRUE;
    bInletValve_Open := FALSE;  // Fail-safe: close valve
END_IF;

// Check for valve faults (stuck open/closed)
IF bValveStuckOpen OR bValveStuckClosed THEN
    bValveFaultDetected := TRUE;
    bLockoutTriggered := TRUE;
    bHighLevelAlarm := TRUE;
    bSystemLockedOut := TRUE;
    bInletValve_Open := FALSE;
END_IF;

// Detect high-level condition only if no sensor fault
IF NOT bSensorFault AND rLevelSensor > HIGH_SETPOINT THEN
    bHighLevelDetected := TRUE;
    bHighLevelAlarm := TRUE;
    bLockoutTriggered := TRUE;
    bInletValve_Open := FALSE;
END_IF;

// Latch lockout state until level drops below reset threshold and manual reset is pressed
IF bLockoutTriggered THEN
    bSystemLockedOut := TRUE;
    bInletValve_Open := FALSE;

    // Reset conditions: level < reset threshold AND manual reset pressed
    IF rLevelSensor <= RESET_THRESHOLD AND bManualReset THEN
        bLockoutTriggered := FALSE;
        bHighLevelAlarm := FALSE;
        bSystemLockedOut := FALSE;
        bInletValve_Open := TRUE;  // Reopen valve after safe reset
    END_IF;
END_IF;
