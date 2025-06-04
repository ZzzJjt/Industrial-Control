(* === Emergency Interlock System for Subsea Gas Wellhead === *)

VAR_INPUT
    PT_101 : REAL;         // Pressure Transmitter (psi)
    TT_101 : REAL;         // Temperature Transmitter (°C)
    FT_101 : REAL;         // Flow Transmitter (m3/h)
    RESET  : BOOL;         // Manual reset command
END_VAR

VAR_OUTPUT
    MV_101      : BOOL := TRUE;   // Master Valve: TRUE = Open, FALSE = Closed
    SHUTDOWN    : BOOL := FALSE;  // Emergency shutdown flag
    Alarm_HighP : BOOL := FALSE;  // Pressure alarm
    Alarm_LowF  : BOOL := FALSE;  // Flow alarm
    Alarm_HighT : BOOL := FALSE;  // Temperature alarm
END_VAR

VAR
    ManualLockout : BOOL := FALSE;  // Lockout prevents auto restart
END_VAR

// === Emergency Conditions ===

IF NOT ManualLockout THEN
    IF PT_101 > 1500.0 THEN
        MV_101 := FALSE;
        SHUTDOWN := TRUE;
        Alarm_HighP := TRUE;
        ManualLockout := TRUE;
    END_IF;

    IF FT_101 < 1.0 THEN // Adjust minimum flow threshold as needed
        MV_101 := FALSE;
        SHUTDOWN := TRUE;
        Alarm_LowF := TRUE;
        ManualLockout := TRUE;
    END_IF;

    IF TT_101 > 120.0 THEN
        MV_101 := FALSE;
        SHUTDOWN := TRUE;
        Alarm_HighT := TRUE;
        ManualLockout := TRUE;
    END_IF;
END_IF;

// === Manual Reset Logic ===

IF RESET = TRUE THEN
    MV_101 := TRUE;
    SHUTDOWN := FALSE;
    Alarm_HighP := FALSE;
    Alarm_LowF := FALSE;
    Alarm_HighT := FALSE;
    ManualLockout := FALSE;
END_IF;
