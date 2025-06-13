PROGRAM SAFETY_DOORS_INTERLOCK
VAR_INPUT
    Execute : BOOL;              // Starts the interlock checks
    DOOR_1_CLOSED : BOOL;        // Digital input indicating if Door 1 is closed
    DOOR_2_CLOSED : BOOL;        // Digital input indicating if Door 2 is closed
    DOOR_3_CLOSED : BOOL;        // Digital input indicating if Door 3 is closed
    ReactorRunning : BOOL;       // Indicates if the reactor is currently running
    ManualReset : BOOL;          // Manual reset input after shutdown due to door opening
END_VAR

VAR_OUTPUT
    ALLOW_START : BOOL;           // Indicates if the reactor can start
    EMERGENCY_SHUTDOWN : BOOL;    // Indicates if an emergency shutdown has been initiated
    Heater_Shutdown : BOOL;       // Indicates if the heater should be shut down
    Agitator_Shutdown : BOOL;     // Indicates if the agitator should be shut down
    Alarm_Triggered : BOOL;      // Indicates if an alarm is triggered
    Error : BOOL;                 // General error flag
    ErrorID : DWORD;              // Detailed error code
END_VAR

VAR
    lastExecute : BOOL := FALSE;  // Last state of the Execute input
    lastManualReset : BOOL := FALSE; // Last state of the Manual Reset input
    AllDoorsClosed : BOOL;        // Combined status of all doors being closed
END_VAR

METHOD Execute : BOOL
BEGIN
    IF R_TRIG(lastExecute, Execute).Q THEN
        // Reset outputs except latched states
        ALLOW_START := FALSE;
        Heater_Shutdown := FALSE;
        Agitator_Shutdown := FALSE;
        Alarm_Triggered := FALSE;
        Error := FALSE;
        ErrorID := 0;

        // Check if all doors are closed
        AllDoorsClosed := DOOR_1_CLOSED AND DOOR_2_CLOSED AND DOOR_3_CLOSED;

        // Prevent reactor startup if any door is open
        IF NOT AllDoorsClosed THEN
            ALLOW_START := FALSE;
            Error := TRUE;
            ErrorID := 1; // One or more doors are open
        ELSE
            ALLOW_START := TRUE;
        END_IF;

        // Immediate shutdown if any door is opened during operation
        IF ReactorRunning AND NOT AllDoorsClosed THEN
            EMERGENCY_SHUTDOWN := TRUE;
            ReactorRunning := FALSE;
            Heater_Shutdown := TRUE;
            Agitator_Shutdown := TRUE;
            Alarm_Triggered := TRUE;
            Error := TRUE;
            ErrorID := 2; // Door opened during operation
        END_IF;

        // Latching mechanism to hold the system in a safe state
        IF EMERGENCY_SHUTDOWN THEN
            Heater_Shutdown := TRUE;
            Agitator_Shutdown := TRUE;
            Alarm_Triggered := TRUE;

            // Allow manual reset only if all doors are closed
            IF R_TRIG(lastManualReset, ManualReset).Q AND AllDoorsClosed THEN
                EMERGENCY_SHUTDOWN := FALSE;
                Alarm_Triggered := FALSE;
                Error := FALSE;
                ErrorID := 0;
            END_IF;
        END_IF;

        // Fail-safe defaults if sensor input is lost or invalid
        IF NOT DOOR_1_CLOSED OR NOT DOOR_2_CLOSED OR NOT DOOR_3_CLOSED THEN
            ALLOW_START := FALSE;
            EMERGENCY_SHUTDOWN := TRUE;
            Heater_Shutdown := TRUE;
            Agitator_Shutdown := TRUE;
            Alarm_Triggered := TRUE;
            Error := TRUE;
            ErrorID := 3; // Sensor input lost or invalid
        END_IF;
    END_IF;

    lastExecute := Execute;
    lastManualReset := ManualReset;
    RETURN TRUE;
END_METHOD

// Main execution loop
Execute();



