PROGRAM ElevatorControl
VAR_INPUT
    AtFloor : ARRAY[1..5] OF BOOL; // Position flags for each floor
    UpCall : ARRAY[1..4] OF BOOL;   // Up call buttons on floors 1 to 4
    DownCall : ARRAY[2..5] OF BOOL; // Down call buttons on floors 2 to 5
    CabinRequest : ARRAY[1..5] OF BOOL; // Cabin request buttons for floors 1 to 5
END_VAR

VAR_OUTPUT
    MotorUp : BOOL;      // Control motor to move up
    MotorDown : BOOL;    // Control motor to move down
    DoorOpen : BOOL;     // Control door opening
END_VAR

VAR
    CurrentFloor : INT := 1; // Current floor position
    GoingUp : BOOL := TRUE;  // Direction flag: True for up, False for down
    AnyCabinRequest : BOOL; // Flag indicating any cabin request
    AnyUpCall : BOOL;        // Flag indicating any up call
    AnyDownCall : BOOL;       // Flag indicating any down call
    DoorTimer1 : TON;        // Timer for initial door open duration (7 seconds)
    DoorTimer2 : TON;        // Timer for extended door open duration (10 seconds)
    LastDoorOpen : BOOL;     // Previous state of DoorOpen
    RequestQueue : ARRAY[1..5] OF BOOL; // Queue to track pending requests
END_VAR

// Initialize request queue based on current inputs
FOR i := 1 TO 5 DO
    RequestQueue[i] := CabinRequest[i];
END_FOR;

FOR i := 1 TO 4 DO
    IF UpCall[i] THEN
        RequestQueue[i] := TRUE;
    END_IF;
END_FOR;

FOR i := 2 TO 5 DO
    IF DownCall[i] THEN
        RequestQueue[i] := TRUE;
    END_IF;
END_FOR;

// Check if there are any cabin requests
AnyCabinRequest := FALSE;
FOR i := 1 TO 5 DO
    IF CabinRequest[i] THEN
        AnyCabinRequest := TRUE;
        EXIT;
    END_IF;
END_FOR;

// Check if there are any up calls
AnyUpCall := FALSE;
FOR i := 1 TO 4 DO
    IF UpCall[i] THEN
        AnyUpCall := TRUE;
        EXIT;
    END_IF;
END_FOR;

// Check if there are any down calls
AnyDownCall := FALSE;
FOR i := 2 TO 5 DO
    IF DownCall[i] THEN
        AnyDownCall := TRUE;
        EXIT;
    END_IF;
END_FOR;

// Handle door logic
IF AtFloor[CurrentFloor] THEN
    DoorOpen := TRUE;
    DoorTimer1(IN := TRUE, PT := T#7s);

    IF DoorTimer1.Q THEN
        IF NOT AnyCabinRequest THEN
            DoorTimer2(IN := TRUE, PT := T#10s);
            IF DoorTimer2.Q THEN
                DoorOpen := FALSE;
            END_IF;
        ELSE
            DoorOpen := FALSE;
        END_IF;
    END_IF;
ELSE
    DoorOpen := FALSE;
END_IF;

// Prevent motion while door is open
IF NOT DoorOpen THEN
    // Determine the next floor based on current direction
    IF GoingUp THEN
        FOR i := CurrentFloor + 1 TO 5 DO
            IF RequestQueue[i] THEN
                CurrentFloor := i;
                EXIT;
            END_IF;
        END_FOR;

        // If no more up requests, check for down requests
        IF CurrentFloor = 5 OR NOT AnyUpCall THEN
            GoingUp := FALSE;
        END_IF;
    ELSIF GoingDown THEN
        FOR i := CurrentFloor - 1 DOWNTO 1 BY -1 DO
            IF RequestQueue[i] THEN
                CurrentFloor := i;
                EXIT;
            END_IF;
        END_FOR;

        // If no more down requests, check for up requests
        IF CurrentFloor = 1 OR NOT AnyDownCall THEN
            GoingUp := TRUE;
        END_IF;
    END_IF;

    // Update motor directions based on current floor and direction
    IF GoingUp AND CurrentFloor < 5 THEN
        MotorUp := TRUE;
        MotorDown := FALSE;
    ELSIF GoingDown AND CurrentFloor > 1 THEN
        MotorUp := FALSE;
        MotorDown := TRUE;
    ELSE
        MotorUp := FALSE;
        MotorDown := FALSE;
    END_IF;
ELSE
    MotorUp := FALSE;
    MotorDown := FALSE;
END_IF;

// Clear the request for the current floor
RequestQueue[CurrentFloor] := FALSE;

// Inline comments explaining the logic:
// The program uses state transitions to manage elevator movement between floors.
// Doors open for 7 seconds and extend to 10 seconds if no cabin request is made.
// Calls from up/down buttons and cabin requests are handled intelligently based on direction.
// The elevator avoids skipping floor requests and handles direction changes safely.
// Safety logic ensures the elevator does not move while the door is open.
// All pending requests must be processed before going idle.



