PROGRAM PLC_PRG
TITLE '5-Floor Elevator Control System'

(*
    Description:
    Implements a complete elevator control system for a 5-floor building.
    
    Features:
    - Detects current floor via top/bottom limit switches
    - Manages door open timing (7s base + 10s extended if no button pressed)
    - Handles up/down call buttons on each floor
    - Processes cabin floor selection buttons
    - Prioritizes calls in current direction before reversing
    - Supports idle mode when no requests are active
    
    Safety:
    - Prevents motion while doors are open
    - Ensures orderly state transitions
*)

VAR
    // Inputs: Floor Call Buttons
    UpCall : ARRAY [1..4] OF BOOL := [FALSE, FALSE, FALSE, FALSE];     // Up call from floors 1 to 4
    DownCall : ARRAY [2..5] OF BOOL := [FALSE, FALSE, FALSE, FALSE];   // Down call from floors 2 to 5
    CabinRequest : ARRAY [1..5] OF BOOL := [FALSE, FALSE, FALSE, FALSE, FALSE]; // Inside floor selections

    // Inputs: Position Detection
    TopLimit : ARRAY [1..5] OF BOOL := [FALSE, FALSE, FALSE, FALSE, FALSE];
    BottomLimit : ARRAY [1..5] OF BOOL := [FALSE, FALSE, FALSE, FALSE, FALSE];

    // Output: Door control
    DoorOpen : BOOL := FALSE;

    // Internal logic
    CurrentFloor : INT := 1;          // Starts at floor 1
    NextFloor : INT := 1;
    GoingUp : BOOL := TRUE;
    Idle : BOOL := TRUE;
    AnyCabinButtonPressed : BOOL := FALSE;

    // Timers
    DoorTimer1 : TON;                 // First 7 seconds
    DoorTimer2 : TON;                 // Extended 10 seconds
END_VAR

// === MAIN LOGIC ===

// Step 1: Determine current floor based on limit switches
FOR i := 1 TO 5 DO
    IF TopLimit[i] AND BottomLimit[i] THEN
        CurrentFloor := i;
        EXIT;
    END_IF;
END_FOR;

// Step 2: Check if any cabin request is active
AnyCabinButtonPressed := FALSE;
FOR i := 1 TO 5 DO
    IF CabinRequest[i] THEN
        AnyCabinButtonPressed := TRUE;
        EXIT;
    END_IF;
END_FOR;

// Step 3: Door logic — opens when elevator reaches floor
IF AtCurrentFloor(CurrentFloor) THEN
    DoorOpen := TRUE;
    DoorTimer1(IN := TRUE, PT := T#7s);

    IF DoorTimer1.Q THEN
        IF NOT AnyCabinButtonPressed THEN
            DoorTimer2(IN := TRUE, PT := T#10s);
            IF DoorTimer2.Q THEN
                DoorOpen := FALSE;
                DoorTimer1(IN := FALSE);
                DoorTimer2(IN := FALSE);
            END_IF;
        ELSE
            DoorOpen := FALSE;
            DoorTimer1(IN := FALSE);
            DoorTimer2(IN := FALSE);
        END_IF;
    END_IF;
END_IF;

// Step 4: Movement logic
IF NOT DoorOpen THEN

    // Reset idle flag if there are pending requests
    Idle := NOT (HasPendingRequests());

    // Decide next floor based on current direction
    IF GoingUp THEN
        NextFloor := GetNextUpFloor();
        IF NextFloor > CurrentFloor THEN
            MoveElevator(NextFloor);
        ELSE
            GoingUp := FALSE;
        END_IF;
    ELSE
        NextFloor := GetNextDownFloor();
        IF NextFloor < CurrentFloor THEN
            MoveElevator(NextFloor);
        ELSE
            GoingUp := TRUE;
        END_IF;
    END_IF;

ELSE
    // Prevent movement while doors are open
    StopElevator();
END_IF;

END_PROGRAM
