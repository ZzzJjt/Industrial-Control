FUNCTION_BLOCK SafetyDoorInterlock
VAR_INPUT
    DOOR_1_CLOSED : BOOL; // Status indicating if Door 1 is closed
    DOOR_2_CLOSED : BOOL; // Status indicating if Door 2 is closed
    START_COMMAND : BOOL; // Start command issued by operator
END_VAR

VAR_OUTPUT
    ALLOW_START : BOOL; // Allow reactor start
    EMERGENCY_SHUTDOWN : BOOL; // Command to perform emergency shutdown
    REACTOR_RUNNING : BOOL; // Indicate if reactor is running
    HEATER_OFF : BOOL; // Command to turn off heater
    STIRRER_OFF : BOOL; // Command to turn off stirrer
    PUMP_OFF : BOOL; // Command to turn off pump
    INTERLOCK_RESET : BOOL; // Command to reset interlock manually
END_VAR

VAR
    EmergencyLatch : BOOL;
    DoorsClosedAtStart : BOOL;
END_VAR

// Main execution logic
ALLOW_START := FALSE;
EMERGENCY_SHUTDOWN := FALSE;
HEATER_OFF := FALSE;
STIRRER_OFF := FALSE;
PUMP_OFF := FALSE;

// Check if all doors are closed at the start attempt
IF START_COMMAND THEN
    IF DOOR_1_CLOSED AND DOOR_2_CLOSED THEN
        ALLOW_START := TRUE;
        REACTOR_RUNNING := TRUE;
        DoorsClosedAtStart := TRUE;
    ELSE
        ALLOW_START := FALSE;
        REACTOR_RUNNING := FALSE;
        DoorsClosedAtStart := FALSE;
    END_IF;
END_IF;

// Monitor door statuses during operation
IF REACTOR_RUNNING THEN
    IF NOT DOOR_1_CLOSED OR NOT DOOR_2_CLOSED THEN
        EMERGENCY_SHUTDOWN := TRUE;
        REACTOR_RUNNING := FALSE;
        EmergencyLatch := TRUE;
        HEATER_OFF := TRUE;
        STIRRER_OFF := TRUE;
        PUMP_OFF := TRUE;
    END_IF;
END_IF;

// Latch the emergency shutdown state until manual reset
IF EmergencyLatch THEN
    EMERGENCY_SHUTDOWN := TRUE;
    HEATER_OFF := TRUE;
    STIRRER_OFF := TRUE;
    PUMP_OFF := TRUE;
    ALLOW_START := FALSE;
END_IF;

// Manual reset of interlock system
IF INTERLOCK_RESET THEN
    EmergencyLatch := FALSE;
END_IF;

// Ensure no process starts if emergency latch is active
IF EmergencyLatch THEN
    ALLOW_START := FALSE;
END_IF;



