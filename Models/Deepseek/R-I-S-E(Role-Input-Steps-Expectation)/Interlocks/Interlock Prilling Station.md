FUNCTION_BLOCK FB_PrillStationInterlock
VAR_INPUT
    rTT_101: REAL;              // Temperature in prill tower
    rFT_102: REAL;              // Cooling air flow
    bPumpTrip: BOOL;            // Melt pump trip signal
    bLS_103: BOOL;              // High-level switch at base
    bScrubberFault: BOOL;       // Scrubber system failure
    rPT_104: REAL;              // Inert gas pressure
    bEStopPressed: BOOL;        // Emergency stop activation
    rPT_105: REAL;              // Feed line pressure
    bPowerOK: BOOL;             // Power supply status
    bDustAlarm: BOOL;           // Dust buildup detected
    bManualReset: BOOL;         // Manual reset after fault clearance
END_VAR

VAR_OUTPUT
    bMV_101_Open: BOOL := FALSE;
    bPump_101_Run: BOOL := FALSE;
    bEV_101_Open: BOOL := FALSE;
    bSystemShutdown: BOOL := FALSE;
    bAlarmActive: BOOL := FALSE;
END_VAR

VAR
    HIGH_TEMP_LIMIT AT %I0.0: REAL := 175.0;
    RESET_TEMP_LIMIT AT %I4.0: REAL := 165.0;
    MIN_AIR_FLOW AT %I8.0: REAL := 80.0;
    MAX_PRESSURE AT %I12.0: REAL := 15.0;
    MIN_N2_PRESSURE AT %I16.0: REAL := 5.0;

    bLockoutState: BOOL := FALSE;
END_VAR

// --- INITIALIZATION ---
bMV_101_Open := FALSE;
bPump_101_Run := FALSE;
bEV_101_Open := FALSE;
bSystemShutdown := FALSE;
bAlarmActive := FALSE;

// --- EMERGENCY STOP ---
IF bEStopPressed THEN
    bLockoutState := TRUE;
    bSystemShutdown := TRUE;
    bAlarmActive := TRUE;
END_IF;

// --- HIGH TEMPERATURE ---
IF rTT_101 > HIGH_TEMP_LIMIT THEN
    bLockoutState := TRUE;
    bSystemShutdown := TRUE;
    bAlarmActive := TRUE;
    bEV_101_Open := TRUE;
END_IF;

// --- COOLING AIR FLOW FAILURE ---
IF rFT_102 < MIN_AIR_FLOW THEN
    bLockoutState := TRUE;
    bAlarmActive := TRUE;
END_IF;

// --- MELT PUMP FAILURE ---
IF bPumpTrip THEN
    bLockoutState := TRUE;
    bAlarmActive := TRUE;
END_IF;

// --- PRODUCT ACCUMULATION ---
IF bLS_103 THEN
    bLockoutState := TRUE;
    bAlarmActive := TRUE;
END_IF;

// --- SCRUBBER FAILURE ---
IF bScrubberFault THEN
    bLockoutState := TRUE;
    bAlarmActive := TRUE;
END_IF;

// --- INERT GAS LOSS ---
IF rPT_104 < MIN_N2_PRESSURE THEN
    bLockoutState := TRUE;
    bAlarmActive := TRUE;
END_IF;

// --- OVERPRESSURE ---
IF rPT_105 > MAX_PRESSURE THEN
    bLockoutState := TRUE;
    bEV_101_Open := TRUE;
    bAlarmActive := TRUE;
END_IF;

// --- POWER FAILURE ---
IF NOT bPowerOK THEN
    bLockoutState := TRUE;
    bAlarmActive := TRUE;
END_IF;

// --- DUST BUILDUP ---
IF bDustAlarm THEN
    bLockoutState := TRUE;
    bAlarmActive := TRUE;
END_IF;

// --- LOCKOUT LOGIC ---
IF bLockoutState THEN
    bMV_101_Open := FALSE;
    bPump_101_Run := FALSE;

    IF bManualReset AND rTT_101 <= RESET_TEMP_LIMIT THEN
        bLockoutState := FALSE;
        bSystemShutdown := FALSE;
        bAlarmActive := FALSE;
        bEV_101_Open := FALSE;
    END_IF;
END_IF;
