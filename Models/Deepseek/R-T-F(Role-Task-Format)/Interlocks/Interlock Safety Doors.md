PROGRAM ReactorDoorInterlocks
VAR_INPUT
    DOOR_1_CLOSED : BOOL; // Door 1 status (TRUE = Closed, FALSE = Open)
    DOOR_2_CLOSED : BOOL; // Door 2 status (TRUE = Closed, FALSE = Open)
    SENSOR_FAULT : BOOL;  // Sensor fault flag (TRUE = Faulty, FALSE = OK)
END_VAR

VAR_OUTPUT
    ALLOW_START : BOOL;           // Allow reactor start (TRUE = Allowed, FALSE = Blocked)
    EMERGENCY_SHUTDOWN : BOOL;   // Emergency shutdown command (TRUE = Shutdown, FALSE = Normal)
    HEATING_STOP : BOOL;         // Command to stop heating process
    MIXING_STOP : BOOL;          // Command to stop mixing process
    PRESSURIZATION_STOP : BOOL;  // Command to stop pressurization process
END_VAR

VAR
    REACTOR_RUNNING : BOOL;       // Indicates if the reactor is currently running
    SHUTDOWN_LATCH : BOOL;        // Latch for emergency shutdown condition
    START_REQUESTED : BOOL;       // Start request from operator
    RESET_REQUESTED : BOOL;       // Reset request from operator
END_VAR

METHOD Execute : BOOL
BEGIN
    // Initialize outputs if not already initialized
    IF NOT SHUTDOWN_LATCH THEN
        ALLOW_START := TRUE;
        EMERGENCY_SHUTDOWN := FALSE;
        HEATING_STOP := FALSE;
        MIXING_STOP := FALSE;
        PRESSURIZATION_STOP := FALSE;
        REACTOR_RUNNING := FALSE;
    END_IF;

    // Check for sensor fault
    IF SENSOR_FAULT THEN
        EMERGENCY_SHUTDOWN := TRUE;
        SHUTDOWN_LATCH := TRUE;
        HEATING_STOP := TRUE;
        MIXING_STOP := TRUE;
        PRESSURIZATION_STOP := TRUE;
        ALLOW_START := FALSE;
        RETURN TRUE;
    END_IF;

    // Check if any door is open
    IF NOT DOOR_1_CLOSED OR NOT DOOR_2_CLOSED THEN
        ALLOW_START := FALSE;
        IF REACTOR_RUNNING THEN
            EMERGENCY_SHUTDOWN := TRUE;
            SHUTDOWN_LATCH := TRUE;
            HEATING_STOP := TRUE;
            MIXING_STOP := TRUE;
            PRESSURIZATION_STOP := TRUE;
        END_IF;
    ELSE
        ALLOW_START := TRUE;
    END_IF;

    // Handle start request
    IF START_REQUESTED AND ALLOW_START AND NOT REACTOR_RUNNING THEN
        REACTOR_RUNNING := TRUE;
    END_IF;

    // Handle reset request
    IF RESET_REQUESTED AND ALL([DOOR_1_CLOSED, DOOR_2_CLOSED]) AND SENSOR_FAULT = FALSE THEN
        SHUTDOWN_LATCH := FALSE;
        EMERGENCY_SHUTDOWN := FALSE;
        HEATING_STOP := FALSE;
        MIXING_STOP := FALSE;
        PRESSURIZATION_STOP := FALSE;
        REACTOR_RUNNING := FALSE;
    END_IF;

    RETURN TRUE;
END_METHOD

END_PROGRAM
