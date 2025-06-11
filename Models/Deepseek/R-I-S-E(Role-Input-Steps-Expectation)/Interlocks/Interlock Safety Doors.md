FUNCTION_BLOCK FB_DoorInterlockSystem
VAR_INPUT
    // Door status inputs – TRUE = Closed
    DOOR_1_CLOSED: BOOL := FALSE;
    DOOR_2_CLOSED: BOOL := FALSE;
    DOOR_3_CLOSED: BOOL := FALSE;

    // Operator input
    START_BUTTON: BOOL := FALSE;
    RESET_BUTTON: BOOL := FALSE;

    // Reactor operational state feedback
    REACTOR_RUNNING: BOOL := FALSE;
END_VAR

VAR_OUTPUT
    // Outputs to control system
    ALLOW_START: BOOL := FALSE;
    EMERGENCY_SHUTDOWN: BOOL := FALSE;
    REACTOR_ENABLE: BOOL := FALSE;
    ALARM_ACTIVE: BOOL := FALSE;
END_VAR

VAR
    // Internal Flags
    bAllDoorsClosed: BOOL := FALSE;
    bShutdownLatched: BOOL := FALSE;
END_VAR

// Evaluate current door status
bAllDoorsClosed := DOOR_1_CLOSED AND DOOR_2_CLOSED AND DOOR_3_CLOSED;

// Allow start only when all doors are closed
ALLOW_START := bAllDoorsClosed AND NOT bShutdownLatched;

// --- START REQUEST HANDLING ---
IF START_BUTTON AND ALLOW_START THEN
    REACTOR_ENABLE := TRUE;
END_IF;

// --- SHUTDOWN DURING OPERATION IF DOORS OPEN ---
IF REACTOR_RUNNING AND NOT bAllDoorsClosed THEN
    EMERGENCY_SHUTDOWN := TRUE;
    REACTOR_ENABLE := FALSE;
    ALARM_ACTIVE := TRUE;
END_IF;

// --- LATCHED SHUTDOWN STATE ---
IF EMERGENCY_SHUTDOWN THEN
    bShutdownLatched := TRUE;
    ALARM_ACTIVE := TRUE;
END_IF;

// --- MANUAL RESET AFTER SHUTDOWN ---
IF bShutdownLatched AND RESET_BUTTON AND bAllDoorsClosed THEN
    bShutdownLatched := FALSE;
    EMERGENCY_SHUTDOWN := FALSE;
    ALARM_ACTIVE := FALSE;
END_IF;

PROGRAM PLC_PRG
VAR
    DoorInterlock: FB_DoorInterlockSystem;

    // Simulated or real-world inputs
    Door1Status: BOOL := DI1;        // TRUE = closed
    Door2Status: BOOL := DI2;        // TRUE = closed
    Door3Status: BOOL := DI3;        // TRUE = closed
    StartBtn: BOOL := DI4;           // Momentary pushbutton
    ResetBtn: BOOL := DI5;           // Momentary pushbutton
    ReactorIsRunning: BOOL := M1.0;  // From drive or motor feedback
END_VAR

// Call function block with real-world inputs
DoorInterlock(
    DOOR_1_CLOSED := Door1Status,
    DOOR_2_CLOSED := Door2Status,
    DOOR_3_CLOSED := Door3Status,
    START_BUTTON := StartBtn,
    RESET_BUTTON := ResetBtn,
    REACTOR_RUNNING := ReactorIsRunning
);
