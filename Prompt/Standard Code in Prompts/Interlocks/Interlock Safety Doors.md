```
PROGRAM SAFETY_DOOR_INTERLOCKS
VAR
    // Inputs
    DOOR_1_STATUS: BOOL; // Door 1 Status (CLOSED = TRUE)
    DOOR_2_STATUS: BOOL; // Door 2 Status (CLOSED = TRUE)
    DOOR_3_STATUS: BOOL; // Door 3 Status (CLOSED = TRUE)
    
    // Outputs
    REACTOR_START: BOOL; // Reactor Start Permission (ALLOWED = TRUE)
    EMERGENCY_SHUTDOWN: BOOL; // Emergency Shutdown Status
    
    // Flags
    DOORS_CLOSED: BOOL; // All Doors Closed Flag
BEGIN
    // Check All Doors Status
    DOORS_CLOSED := DOOR_1_STATUS AND DOOR_2_STATUS AND DOOR_3_STATUS;

    // Allow Reactor Start Only When All Doors Are Closed
    REACTOR_START := DOORS_CLOSED;

    // Trigger Emergency Shutdown If Any Door Is Opened During Operation
    IF NOT DOORS_CLOSED THEN
        EMERGENCY_SHUTDOWN := TRUE;
    ELSE
        EMERGENCY_SHUTDOWN := FALSE;
    END_IF;

    // Additional Logic Can Be Added Here To Handle Other Aspects Of Reactor Operation
END
END_PROGRAM
```
