FUNCTION_BLOCK FB_SafetyDoorInterlock
VAR_INPUT
    DOOR_1_CLOSED : BOOL;
    DOOR_2_CLOSED : BOOL;
    DOOR_3_CLOSED : BOOL;
    ReactorStartRequest : BOOL; // Command to start reactor
    ManualReset : BOOL;         // Reset button (rising edge)
END_VAR

VAR_OUTPUT
    ALLOW_START : BOOL;         // TRUE when all doors closed and ready
    EMERGENCY_SHUTDOWN : BOOL;  // Triggers emergency stop
    ReactorRunning : BOOL;      // Status of reactor
END_VAR

VAR
    AllDoorsClosed : BOOL;
    ResetLatch : BOOL := TRUE;
    PrevManualReset : BOOL := FALSE;
    RisingEdgeReset : BOOL;
END_VAR

// Determine door status
AllDoorsClosed := DOOR_1_CLOSED AND DOOR_2_CLOSED AND DOOR_3_CLOSED;
ALLOW_START := AllDoorsClosed AND ResetLatch;

// Detect rising edge on ManualReset
RisingEdgeReset := ManualReset AND NOT PrevManualReset;
PrevManualReset := ManualReset;

// Emergency interlock logic
IF ReactorRunning AND NOT AllDoorsClosed THEN
    EMERGENCY_SHUTDOWN := TRUE;
    ReactorRunning := FALSE;
    ResetLatch := FALSE;
END_IF

// Normal startup logic
IF ALLOW_START AND ReactorStartRequest THEN
    ReactorRunning := TRUE;
END_IF

// Manual reset logic: Only allowed if all doors are closed
IF RisingEdgeReset AND AllDoorsClosed THEN
    EMERGENCY_SHUTDOWN := FALSE;
    ResetLatch := TRUE;
END_IF
