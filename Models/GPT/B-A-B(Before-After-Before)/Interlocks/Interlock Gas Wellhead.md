FUNCTION_BLOCK FB_SubseaWellhead_Interlocks
VAR_INPUT
    PT_101       : REAL;     // Pressure transmitter (psi)
    TT_101       : REAL;     // Temperature transmitter (°C)
    FT_101       : REAL;     // Flow transmitter (SCFM or similar)
    MIN_FLOW     : REAL;     // Minimum allowed flow rate
    ResetManual  : BOOL;     // Manual reset command
END_VAR

VAR_OUTPUT
    MV_101       : BOOL;     // Master valve status (TRUE=open, FALSE=closed)
    SHUTDOWN     : BOOL;     // Emergency shutdown flag
    AlarmHighP   : BOOL;     // High pressure alarm
    AlarmHighT   : BOOL;     // High temperature alarm
    AlarmLowF    : BOOL;     // Low flow alarm
END_VAR

VAR
    ShutdownLatched : BOOL := FALSE;
END_VAR

// --- Interlock Logic ---

AlarmHighP := PT_101 > 1500.0;
AlarmHighT := TT_101 > 120.0;
AlarmLowF  := FT_101 < MIN_FLOW;

// --- Trigger Emergency Shutdown ---

IF AlarmHighP OR AlarmHighT OR AlarmLowF THEN
    ShutdownLatched := TRUE;
END_IF

// --- Maintain Shutdown Until Manual Reset ---

IF ShutdownLatched THEN
    SHUTDOWN := TRUE;
    MV_101 := FALSE; // Close master valve
ELSE
    SHUTDOWN := FALSE;
    MV_101 := TRUE;  // Open only when no faults
END_IF

// --- Manual Reset Logic ---

IF ResetManual AND NOT (AlarmHighP OR AlarmHighT OR AlarmLowF) THEN
    ShutdownLatched := FALSE;
END_IF
