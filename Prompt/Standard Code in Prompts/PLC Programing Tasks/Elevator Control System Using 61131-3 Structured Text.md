```
PROGRAM ElevatorControlSystem
VAR
    CurrentFloor : INT := 1; // Current floor of the elevator (1-5)
    Direction : INT := 0; // Direction of travel (-1 for down, 0 for stop, 1 for up)
    DoorOpen : BOOL := FALSE; // Indicates if the elevator door is open
    DoorOpenTimer : TIME := T#0s; // Timer for the door open period
    DoorReopenTimer : TIME := T#0s; // Timer for the door reopen period
    TopLimit : BOOL := FALSE; // Top limit switch status
    BottomLimit : BOOL := FALSE; // Bottom limit switch status
    CabinButtons : ARRAY [1..5] OF BOOL := (FALSE, FALSE, FALSE, FALSE, FALSE); // Cabin button status
    FloorCallButtons : ARRAY [1..5] OF ARRAY [1..2] OF BOOL := ((FALSE, FALSE), (FALSE, FALSE), (FALSE, FALSE), (FALSE, FALSE), (FALSE, FALSE)); // Up/Down call button status for each floor
END_VAR

// Main control logic
IF TopLimit THEN
    // If the elevator is at the top floor, set direction to stop
    Direction := 0;
ELSIF BottomLimit THEN
    // If the elevator is at the bottom floor, set direction to stop
    Direction := 0;
ELSE
    // Determine the direction based on call buttons and current floor
    IF FloorCallButtons[CurrentFloor][1] OR FloorCallButtons[CurrentFloor][2] THEN
        // If there is a call at the current floor, ignore it if already there
    ELSE
        Direction := FindDirection(FloorCallButtons, CabinButtons, CurrentFloor);
    END_IF;
END_IF;

// Move the elevator
IF Direction = 1 THEN
    // Move up
    CurrentFloor := CurrentFloor + 1;
ELSIF Direction = -1 THEN
    // Move down
    CurrentFloor := CurrentFloor - 1;
END_IF;

// Handle door operations
IF DoorOpen THEN
    // Door is open
    DoorOpenTimer := DoorOpenTimer + T#1s;
    IF DoorOpenTimer >= T#7s THEN
        // If the door has been open for 7 seconds
        DoorOpen := FALSE;
        DoorOpenTimer := T#0s;
        IF NOT (CabinButtons[CurrentFloor] OR CabinButtons[CurrentFloor+1] OR CabinButtons[CurrentFloor+2] OR CabinButtons[CurrentFloor+3] OR CabinButtons[CurrentFloor+4]) THEN
            // If no cabin buttons are pressed, reopen the door for 10 seconds
            DoorReopenTimer := T#0s;
            DoorOpen := TRUE;
        END_IF;
    END_IF;
ELSIF DoorReopenTimer >= T#10s THEN
    // If the door has been reopened for 10 seconds, close the door
    DoorOpen := FALSE;
    DoorReopenTimer := T#0s;
END_IF;

// Open the door when the elevator stops at a floor
IF CurrentFloor MOD 1 = 0 THEN
    // If the elevator has stopped at a floor
    DoorOpen := TRUE;
    DoorOpenTimer := T#0s;
    DoorReopenTimer := T#0s;
END_IF;

// Debugging outputs (for simulation purposes)
// Example: Write("Current Floor", CurrentFloor);
// Example: Write("Direction", Direction);
// Example: Write("Door Open", DoorOpen);
// Example: Write("Door Open Timer", DoorOpenTimer);
// Example: Write("Top Limit", TopLimit);
// Example: Write("Bottom Limit", BottomLimit);
// Example: Write("Cabin Buttons", CabinButtons);
// Example: Write("Floor Call Buttons", FloorCallButtons);

END_PROGRAM

FUNCTION FindDirection : INT (FloorCallButtons : ARRAY [1..5] OF ARRAY [1..2] OF BOOL; CabinButtons : ARRAY [1..5] OF BOOL; CurrentFloor : INT)
VAR
    NextFloorUp : INT := CurrentFloor + 1; // Next floor going up
    NextFloorDown : INT := CurrentFloor - 1; // Next floor going down
    UpCallActive : BOOL := FALSE; // Up call button active at the next floor
    DownCallActive : BOOL := FALSE; // Down call button active at the next floor
    CabinCallActive : BOOL := FALSE; // Cabin button active for the next floor
END_VAR

// Determine if there is a call to go up or down
UpCallActive := FloorCallButtons[NextFloorUp][1];
DownCallActive := FloorCallButtons[NextFloorDown][2];
CabinCallActive := CabinButtons[NextFloorUp];

// Set direction based on calls and current floor
IF UpCallActive OR CabinCallActive THEN
    FindDirection := 1; // Set direction to up
ELSIF DownCallActive THEN
    FindDirection := -1; // Set direction to down
ELSE
    FindDirection := 0; // Set direction to stop
END_IF;

END_FUNCTION
```
