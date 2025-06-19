PROGRAM PLC_PRG
VAR
    // Inputs - Safety Door Status
    DOOR_1_CLOSED: BOOL := TRUE;   // TRUE = Closed, FALSE = Open
    DOOR_2_CLOSED: BOOL := TRUE;
    DOOR_3_CLOSED: BOOL := TRUE;

    // Inputs - Operator Controls
    START_BUTTON: BOOL := FALSE;
    RESET_BUTTON: BOOL := FALSE;

    // Internal Flags
    ALL_DOORS_CLOSED: BOOL := TRUE;
    DOORS_CHECKED_AT_STARTUP: BOOL := FALSE;
    EMERGENCY_SHUTDOWN: BOOL := FALSE;
    SYSTEM_RESET: BOOL := FALSE;

    // Outputs - Reactor Operation
    ReactorRunning: BOOL := FALSE;
    Heater_ON: BOOL := FALSE;
    Stirrer_ON: BOOL := FALSE;
    FeedPump_ON: BOOL := FALSE;

    // Alarms
    DoorOpenAlarm: BOOL := FALSE;
END_VAR

// --- Door Monitoring Logic ---
ALL_DOORS_CLOSED := DOOR_1_CLOSED AND DOOR_2_CLOSED AND DOOR_3_CLOSED;

// --- Startup Interlock Logic ---
IF NOT DOORS_CHECKED_AT_STARTUP THEN
    IF NOT ALL_DOORS_CLOSED THEN
        DoorOpenAlarm := TRUE;
        ALLOW_START := FALSE;
    ELSE
        DoorOpenAlarm := FALSE;
        DOORS_CHECKED_AT_STARTUP := TRUE;
    END_IF;
END_IF;

// --- Emergency Shutdown on Door Opening During Operation ---
IF ReactorRunning AND NOT ALL_DOORS_CLOSED THEN
    EMERGENCY_SHUTDOWN := TRUE;
    ReactorRunning := FALSE;
    DoorOpenAlarm := TRUE;
END_IF;

// --- Reset Logic ---
IF RESET_BUTTON AND EMERGENCY_SHUTDOWN THEN
    IF ALL_DOORS_CLOSED THEN
        EMERGENCY_SHUTDOWN := FALSE;
        SYSTEM_RESET := TRUE;
        DoorOpenAlarm := FALSE;
    END_IF;
END_IF;

// --- Reactor Start Logic ---
IF START_BUTTON AND ALL_DOORS_CLOSED AND SYSTEM_RESET AND NOT EMERGENCY_SHUTDOWN THEN
    ReactorRunning := TRUE;
    Heater_ON := TRUE;
    Stirrer_ON := TRUE;
    FeedPump_ON := TRUE;
    SYSTEM_RESET := FALSE;
    DOORS_CHECKED_AT_STARTUP := FALSE;
END_IF;

// --- Emergency Shutdown Actions ---
IF EMERGENCY_SHUTDOWN THEN
    Heater_ON := FALSE;
    Stirrer_ON := FALSE;
    FeedPump_ON := FALSE;
END_IF;
