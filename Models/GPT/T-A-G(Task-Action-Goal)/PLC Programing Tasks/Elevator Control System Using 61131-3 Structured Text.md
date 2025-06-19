// IEC 61131-3 Structured Text Program for 5-Floor Elevator Control

VAR
    AtFloor : ARRAY[1..5] OF BOOL; // Floor detection
    CabinRequest : ARRAY[1..5] OF BOOL; // Buttons inside elevator
    UpCall : ARRAY[1..4] OF BOOL; // External up buttons (floors 1-4)
    DownCall : ARRAY[2..5] OF BOOL; // External down buttons (floors 2-5)
    TopLimit, BottomLimit : BOOL; // Optional limit switches

    DoorOpen : BOOL := FALSE;
    GoingUp : BOOL := TRUE;
    CurrentFloor : INT := 1;
    MoveUp, MoveDown : BOOL := FALSE;

    Timer7s : TON;
    Timer10s : TON;

    AnyCabinRequest : BOOL;
    AnyRequestAbove : BOOL;
    AnyRequestBelow : BOOL;
END_VAR

// Evaluate all cabin requests
AnyCabinRequest := FALSE;
FOR i := 1 TO 5 DO
    IF CabinRequest[i] THEN
        AnyCabinRequest := TRUE;
    END_IF;
END_FOR

// Evaluate if there are requests above or below the current floor
AnyRequestAbove := FALSE;
FOR i := CurrentFloor + 1 TO 5 DO
    IF (CabinRequest[i] OR UpCall[i - 1]) THEN
        AnyRequestAbove := TRUE;
    END_IF;
END_FOR

AnyRequestBelow := FALSE;
FOR i := 1 TO CurrentFloor - 1 DO
    IF (CabinRequest[i] OR DownCall[i + 1]) THEN
        AnyRequestBelow := TRUE;
    END_IF;
END_FOR

// Door open logic
IF AtFloor[CurrentFloor] THEN
    Timer7s(IN := TRUE, PT := T#7s);
    DoorOpen := TRUE;

    IF Timer7s.Q THEN
        IF NOT AnyCabinRequest THEN
            Timer10s(IN := TRUE, PT := T#10s);
            IF Timer10s.Q THEN
                DoorOpen := FALSE;
            END_IF;
        ELSE
            DoorOpen := FALSE;
        END_IF;
    END_IF;
ELSE
    Timer7s(IN := FALSE);
    Timer10s(IN := FALSE);
END_IF

// Movement logic only if doors are closed
IF NOT DoorOpen THEN
    IF GoingUp THEN
        IF AnyRequestAbove AND (CurrentFloor < 5) THEN
            MoveUp := TRUE;
            MoveDown := FALSE;
        ELSIF AnyRequestBelow THEN
            GoingUp := FALSE;
        ELSE
            MoveUp := FALSE;
            MoveDown := FALSE;
        END_IF;
    ELSE // GoingDown
        IF AnyRequestBelow AND (CurrentFloor > 1) THEN
            MoveDown := TRUE;
            MoveUp := FALSE;
        ELSIF AnyRequestAbove THEN
            GoingUp := TRUE;
        ELSE
            MoveUp := FALSE;
            MoveDown := FALSE;
        END_IF;
    END_IF;
ELSE
    MoveUp := FALSE;
    MoveDown := FALSE;
END_IF
