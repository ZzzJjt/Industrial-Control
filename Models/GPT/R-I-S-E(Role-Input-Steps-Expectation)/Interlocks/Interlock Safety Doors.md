VAR
    DOOR_1_CLOSED : BOOL;
    DOOR_2_CLOSED : BOOL;
    DOOR_3_CLOSED : BOOL;

    ReactorStartRequest : BOOL;        // Operator's request to start reactor
    ReactorRunning : BOOL;            // Actual state of reactor
    EMERGENCY_SHUTDOWN : BOOL;       // Latching shutdown signal
    ResetRequested : BOOL;           // Manual reset command

    ALLOW_START : BOOL;              // Internal flag: true only if all doors are closed
    AllDoorsClosed : BOOL;           // Helper flag
END_VAR

// Evaluate whether all doors are closed
AllDoorsClosed := DOOR_1_CLOSED AND DOOR_2_CLOSED AND DOOR_3_CLOSED;

// Startup permitted only if all doors are closed and no shutdown is active
ALLOW_START := AllDoorsClosed AND NOT EMERGENCY_SHUTDOWN;

// Start reactor only if requested, allowed, and not already running
IF ReactorStartRequest AND ALLOW_START AND NOT ReactorRunning THEN
    ReactorRunning := TRUE;
END_IF;

// Emergency shutdown if any door opens while running
IF ReactorRunning AND NOT AllDoorsClosed THEN
    ReactorRunning := FALSE;
    EMERGENCY_SHUTDOWN := TRUE;
END_IF;

// Manual reset clears emergency latch after all doors are closed
IF EMERGENCY_SHUTDOWN AND ResetRequested AND AllDoorsClosed THEN
    EMERGENCY_SHUTDOWN := FALSE;
END_IF;
