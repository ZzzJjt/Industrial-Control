PROGRAM ElevatorController
VAR
    // Floor and direction states
    CurrentFloor       : INT := 1;            // 1 to 5
    GoingUp            : BOOL := TRUE;
    GoingDown          : BOOL := FALSE;

    // Position detection: TRUE if elevator is at that floor
    AtFloor            : ARRAY[1..5] OF BOOL;

    // Requests
    UpCall             : ARRAY[1..4] OF BOOL; // Floor 1–4
    DownCall           : ARRAY[2..5] OF BOOL; // Floor 2–5
    CabinRequest       : ARRAY[1..5] OF BOOL;

    // Door control
    DoorOpen           : BOOL := FALSE;
    DoorTimer1         : TON; // First open duration (7s)
    DoorTimer2         : TON; // Extended open duration (10s)

    // Motion output
    MoveUp             : BOOL := FALSE;
    MoveDown           : BOOL := FALSE;

    // Internal helper
    AnyCabinRequest    : BOOL;
    AnyCallAbove       : BOOL;
    AnyCallBelow       : BOOL;
    i                  : INT;
END_VAR

// ======== Door Logic ========
IF AtFloor[CurrentFloor] THEN
    DoorOpen := TRUE;
    DoorTimer1(IN := TRUE, PT := T#7s);

    // Check if initial door timer expired
    IF DoorTimer1.Q THEN
        // Check if any cabin request was made during the wait
        AnyCabinRequest := FALSE;
        FOR i := 1 TO 5 DO
            IF CabinRequest[i] THEN
                AnyCabinRequest := TRUE;
            END_IF;
        END_FOR;

        IF NOT AnyCabinRequest THEN
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
ELSE
    DoorTimer1(IN := FALSE);
    DoorTimer2(IN := FALSE);
END_IF

// ======== Request Detection ========
AnyCallAbove := FALSE;
FOR i := CurrentFloor+1 TO 5 DO
    IF (CabinRequest[i] OR (i <= 4 AND UpCall[i])) THEN
        AnyCallAbove := TRUE;
    END_IF;
END_FOR;

AnyCallBelow := FALSE;
FOR i := 1 TO CurrentFloor-1 DO
    IF (CabinRequest[i] OR (i >= 2 AND DownCall[i])) THEN
        AnyCallBelow := TRUE;
    END_IF;
END_FOR;

// ======== Motion Control Logic ========
IF NOT DoorOpen THEN
    MoveUp := FALSE;
    MoveDown := FALSE;

    IF GoingUp THEN
        IF AnyCallAbove THEN
            MoveUp := TRUE;
        ELSIF AnyCallBelow THEN
            GoingUp := FALSE;
            GoingDown := TRUE;
            MoveDown := TRUE;
        END_IF;

    ELSIF GoingDown THEN
        IF AnyCallBelow THEN
            MoveDown := TRUE;
        ELSIF AnyCallAbove THEN
            GoingDown := FALSE;
            GoingUp := TRUE;
            MoveUp := TRUE;
        END_IF;
    END_IF;
ELSE
    // Safety interlock: stop motion while door is open
    MoveUp := FALSE;
    MoveDown := FALSE;
END_IF;

// ======== Arrival at Floor Handling ========
IF AtFloor[CurrentFloor] THEN
    // Clear served requests
    CabinRequest[CurrentFloor] := FALSE;
    IF CurrentFloor >= 2 THEN DownCall[CurrentFloor] := FALSE; END_IF;
    IF CurrentFloor <= 4 THEN UpCall[CurrentFloor] := FALSE; END_IF;

    // Stop movement
    MoveUp := FALSE;
    MoveDown := FALSE;
END_IF;
