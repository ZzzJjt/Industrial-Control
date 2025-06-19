FUNCTION_BLOCK FB_SafetyDoorInterlock
VAR_INPUT
    DOOR_1_CLOSED     : BOOL;
    DOOR_2_CLOSED     : BOOL;
    DOOR_3_CLOSED     : BOOL;
    StartCommand      : BOOL;     // Operator attempts to start reactor
    ManualReset       : BOOL;     // Manual reset button
END_VAR

VAR_OUTPUT
    ReactorRunning    : BOOL;
    EmergencyShutdown : BOOL;
    AllowStart        : BOOL;
    Shutdown_Heater   : BOOL;
    Shutdown_Pump     : BOOL;
    Shutdown_Stirrer  : BOOL;
END_VAR

VAR
    InterlockTripped  : BOOL := FALSE;
END_VAR

// --- Check door status ---
AllowStart := DOOR_1_CLOSED AND DOOR_2_CLOSED AND DOOR_3_CLOSED AND NOT InterlockTripped;

// --- Startup logic ---
IF StartCommand AND AllowStart THEN
    ReactorRunning := TRUE;
END_IF

// --- Runtime interlock check ---
IF ReactorRunning AND (NOT DOOR_1_CLOSED OR NOT DOOR_2_CLOSED OR NOT DOOR_3_CLOSED) THEN
    EmergencyShutdown := TRUE;
    ReactorRunning := FALSE;
    InterlockTripped := TRUE;
END_IF

// --- Force shutdown of hazardous components ---
Shutdown_Heater := NOT ReactorRunning;
Shutdown_Pump := NOT ReactorRunning;
Shutdown_Stirrer := NOT ReactorRunning;

// --- Manual reset logic ---
IF ManualReset AND AllowStart THEN
    EmergencyShutdown := FALSE;
    InterlockTripped := FALSE;
END_IF
