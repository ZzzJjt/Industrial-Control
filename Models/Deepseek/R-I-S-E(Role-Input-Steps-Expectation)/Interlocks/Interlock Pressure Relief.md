FUNCTION_BLOCK FB_PressureReliefInterlock
VAR_INPUT
    rPT_101: REAL;             // Pressure Transmitter Reading – bar
    bSensorFault: BOOL;        // TRUE if PT_101 signal invalid/faulted
    bValveStuckClosed: BOOL;   // TRUE if valve feedback indicates stuck closed
    bValveStuckOpen: BOOL;     // TRUE if valve feedback indicates stuck open
    bManualReset: BOOL;        // Operator-initiated reset after fault clearance
END_VAR

VAR_OUTPUT
    bReliefValve_Open: BOOL;   // Relief Valve Command – TRUE = Open, FALSE = Closed
    bOverpressureAlarm: BOOL;  // Alarm output for SCADA or HMI
    bSystemLockedOut: BOOL;    // Lockout state until manual reset
END_VAR

VAR
    // Configuration Constants
    HIGH_LIMIT AT %I0.0: REAL := 15.0;       // Overpressure trip point (bar)
    RESET_LIMIT AT %I4.0: REAL := 12.0;     // Reset level before closing valve

    // Internal Flags
    bOverpressureDetected: BOOL := FALSE;
    bValveFaultDetected: BOOL := FALSE;
    bLockoutTriggered: BOOL := FALSE;
END_VAR

// Initialize outputs at start of scan
bReliefValve_Open := FALSE;
bOverpressureAlarm := FALSE;
bSystemLockedOut := FALSE;

// --- FAIL-SAFE HANDLING FOR SENSOR OR VALVE FAULTS ---
IF bSensorFault THEN
    // Sensor failure: assume worst-case overpressure
    bOverpressureAlarm := TRUE;
    bLockoutTriggered := TRUE;
    bSystemLockedOut := TRUE;
    bReliefValve_Open := TRUE;  // Fail-safe: open valve
END_IF;

IF bValveStuckClosed OR bValveStuckOpen THEN
    // Valve fault detected – prevent unsafe operation
    bValveFaultDetected := TRUE;
    bOverpressureAlarm := TRUE;
    bLockoutTriggered := TRUE;
    bSystemLockedOut := TRUE;
    bReliefValve_Open := TRUE;  // Ensure safe path for pressure release
END_IF;

// --- OVERPRESSURE DETECTION ---
IF NOT bSensorFault AND rPT_101 > HIGH_LIMIT THEN
    bOverpressureDetected := TRUE;
    bOverpressureAlarm := TRUE;
    bLockoutTriggered := TRUE;
    bReliefValve_Open := TRUE;
END_IF;

// --- LATCHING LOGIC WITH MANUAL RESET ---
IF bLockoutTriggered THEN
    bSystemLockedOut := TRUE;

    // Keep valve open until pressure drops below reset threshold AND manual reset pressed
    IF rPT_101 <= RESET_LIMIT AND bManualReset THEN
        bLockoutTriggered := FALSE;
        bOverpressureAlarm := FALSE;
        bSystemLockedOut := FALSE;
        bReliefValve_Open := FALSE;  // Safe to close valve after reset
    END_IF;
END_IF;
