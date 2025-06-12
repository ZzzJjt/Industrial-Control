PROGRAM PrillingStationInterlocks
VAR_INPUT
    PT_101 : REAL;       // Temperature Transmitter Value (°C)
    PR_101 : REAL;       // Pressure Transmitter Value (bar)
    CF_101 : REAL;       // Cooling Air Flow Sensor Value (m³/min)
    MF_101 : BOOL;       // Melt Pump Status (TRUE = Running, FALSE = Stopped)
    FF_101 : BOOL;       // Feeder Status (TRUE = Running, FALSE = Stopped)
    LS_101 : REAL;       // Level Sensor Value (% Capacity)
    SS_101 : BOOL;       // Scrubber Status (TRUE = Active, FALSE = Failed)
    SB_101 : BOOL;       // Scrubber Bypass Switch (TRUE = Bypassed, FALSE = Normal)
    ES_101 : BOOL;       // Emergency Stop Button (TRUE = Pressed, FALSE = Released)
    IG_101 : REAL;       // Inerting Gas Flow Sensor Value (m³/min)
    PF_101 : BOOL;       // Power Failure Detected (TRUE = Failed, FALSE = OK)
    EV_101 : BOOL;       // Explosion Vent Activated (TRUE = Activated, FALSE = Not Activated)
END_VAR

VAR_OUTPUT
    STOP_MELT_FEED : BOOL; // Command to stop melt feed
    SHUTDOWN_TOWER : BOOL; // Command to shut down prilling tower
    OPEN_VENT_VALVE : BOOL; // Command to open vent valve
    HIGH_PRESSURE_RELIEF : BOOL; // Command to activate high-pressure relief valve
    EMERGENCY_COOLING : BOOL; // Command to activate emergency cooling system
    DIAGNOSTIC_CHECK : BOOL; // Command to initiate diagnostic check
    ALARM : BOOL; // General alarm flag
END_VAR

VAR
    OVERTEMP_THRESHOLD : REAL := 120.0; // Overtemperature threshold in °C
    HIGHPRESS_THRESHOLD : REAL := 5.0; // High pressure threshold in bar
    MIN_COOL_FLOW : REAL := 10.0; // Minimum cooling air flow in m³/min
    MAX_LEVEL : REAL := 90.0; // Maximum level threshold in % capacity
    MIN_INERT_GAS_FLOW : REAL := 5.0; // Minimum inerting gas flow in m³/min
    INTERLOCK_LATCH : ARRAY[1..10] OF BOOL; // Latches for each interlock
END_VAR

METHOD Execute : BOOL
BEGIN
    // Initialize outputs if not already initialized
    IF NOT ANY(INTERLOCK_LATCH) THEN
        STOP_MELT_FEED := FALSE;
        SHUTDOWN_TOWER := FALSE;
        OPEN_VENT_VALVE := FALSE;
        HIGH_PRESSURE_RELIEF := FALSE;
        EMERGENCY_COOLING := FALSE;
        DIAGNOSTIC_CHECK := FALSE;
        ALARM := FALSE;
    END_IF;

    // Overtemperature Interlock
    IF PT_101 > OVERTEMP_THRESHOLD AND NOT INTERLOCK_LATCH[1] THEN
        STOP_MELT_FEED := TRUE;
        SHUTDOWN_TOWER := TRUE;
        OPEN_VENT_VALVE := TRUE;
        ALARM := TRUE;
        INTERLOCK_LATCH[1] := TRUE;
    END_IF;

    // High Pressure Interlock
    IF PR_101 > HIGHPRESS_THRESHOLD AND NOT INTERLOCK_LATCH[2] THEN
        STOP_MELT_FEED := TRUE;
        SHUTDOWN_TOWER := TRUE;
        HIGH_PRESSURE_RELIEF := TRUE;
        ALARM := TRUE;
        INTERLOCK_LATCH[2] := TRUE;
    END_IF;

    // Cooling Air Failure Interlock
    IF CF_101 < MIN_COOL_FLOW AND NOT INTERLOCK_LATCH[3] THEN
        STOP_MELT_FEED := TRUE;
        SHUTDOWN_TOWER := TRUE;
        EMERGENCY_COOLING := TRUE;
        ALARM := TRUE;
        INTERLOCK_LATCH[3] := TRUE;
    END_IF;

    // Melt Pump or Feeder Failure Interlock
    IF NOT MF_101 OR NOT FF_101 AND NOT INTERLOCK_LATCH[4] THEN
        STOP_MELT_FEED := TRUE;
        SHUTDOWN_TOWER := TRUE;
        DIAGNOSTIC_CHECK := TRUE;
        ALARM := TRUE;
        INTERLOCK_LATCH[4] := TRUE;
    END_IF;

    // High Product Accumulation Interlock
    IF LS_101 > MAX_LEVEL AND NOT INTERLOCK_LATCH[5] THEN
        STOP_MELT_FEED := TRUE;
        SHUTDOWN_TOWER := TRUE;
        ALARM := TRUE;
        INTERLOCK_LATCH[5] := TRUE;
    END_IF;

    // Scrubber Failure or Bypass Interlock
    IF NOT SS_101 OR SB_101 AND NOT INTERLOCK_LATCH[6] THEN
        SHUTDOWN_TOWER := TRUE;
        ALARM := TRUE;
        INTERLOCK_LATCH[6] := TRUE;
    END_IF;

    // Emergency Stop Interlock
    IF ES_101 AND NOT INTERLOCK_LATCH[7] THEN
        STOP_MELT_FEED := TRUE;
        SHUTDOWN_TOWER := TRUE;
        OPEN_VENT_VALVE := TRUE;
        ALARM := TRUE;
        INTERLOCK_LATCH[7] := TRUE;
    END_IF;

    // Loss of Inerting Gas Interlock
    IF IG_101 < MIN_INERT_GAS_FLOW AND NOT INTERLOCK_LATCH[8] THEN
        STOP_MELT_FEED := TRUE;
        SHUTDOWN_TOWER := TRUE;
        ALARM := TRUE;
        INTERLOCK_LATCH[8] := TRUE;
    END_IF;

    // Power Failure Interlock
    IF PF_101 AND NOT INTERLOCK_LATCH[9] THEN
        STOP_MELT_FEED := TRUE;
        SHUTDOWN_TOWER := TRUE;
        OPEN_VENT_VALVE := TRUE;
        ALARM := TRUE;
        INTERLOCK_LATCH[9] := TRUE;
    END_IF;

    // Explosion Vent Activation Interlock
    IF EV_101 AND NOT INTERLOCK_LATCH[10] THEN
        SHUTDOWN_TOWER := TRUE;
        ALARM := TRUE;
        INTERLOCK_LATCH[10] := TRUE;
    END_IF;

    // Reset Logic
    IF NOT ANY([SensorFault, ValveFault]) THEN
        IF PT_101 <= OVERTEMP_THRESHOLD THEN
            INTERLOCK_LATCH[1] := FALSE;
        END_IF;
        IF PR_101 <= HIGHPRESS_THRESHOLD THEN
            INTERLOCK_LATCH[2] := FALSE;
        END_IF;
        IF CF_101 >= MIN_COOL_FLOW THEN
            INTERLOCK_LATCH[3] := FALSE;
        END_IF;
        IF MF_101 AND FF_101 THEN
            INTERLOCK_LATCH[4] := FALSE;
        END_IF;
        IF LS_101 <= MAX_LEVEL THEN
            INTERLOCK_LATCH[5] := FALSE;
        END_IF;
        IF SS_101 AND NOT SB_101 THEN
            INTERLOCK_LATCH[6] := FALSE;
        END_IF;
        IF NOT ES_101 THEN
            INTERLOCK_LATCH[7] := FALSE;
        END_IF;
        IF IG_101 >= MIN_INERT_GAS_FLOW THEN
            INTERLOCK_LATCH[8] := FALSE;
        END_IF;
        IF NOT PF_101 THEN
            INTERLOCK_LATCH[9] := FALSE;
        END_IF;
        IF NOT EV_101 THEN
            INTERLOCK_LATCH[10] := FALSE;
        END_IF;
    END_IF;

    RETURN TRUE;
END_METHOD

END_PROGRAM
