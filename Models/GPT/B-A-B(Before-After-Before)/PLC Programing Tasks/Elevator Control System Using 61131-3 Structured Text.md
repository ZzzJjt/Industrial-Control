// === Inputs ===
VAR
    AtFloor : ARRAY[1..5] OF BOOL;             // Position sensors
    UpCall : ARRAY[1..4] OF BOOL;              // Floor up-call buttons (1–4)
    DownCall : ARRAY[2..5] OF BOOL;            // Floor down-call buttons (2–5)
    CabinRequest : ARRAY[1..5] OF BOOL;        // Inside cabin floor requests
    AnyCabinButtonPressed : BOOL;              // TRUE if any CabinRequest[] is active
END_VAR

// === Outputs ===
VAR
    MoveUp, MoveDown, DoorOpen : BOOL;
    CurrentFloor : INT := 1;
    Idle, GoingUp : BOOL := TRUE;
END_VAR

// === Internal State ===
VAR
    DoorTimer1 : TON;
    DoorTimer2 : TON;
    i : INT;
    RequestsAbove, RequestsBelow : BOOL;
END_VAR

// === Door Control Logic ===
IF AtFloor[CurrentFloor] THEN
    DoorOpen := TRUE;
    DoorTimer1(IN := TRUE, PT := T#7s);

    IF DoorTimer1.Q THEN
        IF NOT AnyCabinButtonPressed THEN
            DoorTimer2(IN := TRUE, PT := T#10s);
            IF DoorTimer2.Q THEN
                DoorOpen := FALSE;
            END_IF;
        ELSE
            DoorOpen := FALSE;
        END_IF;
    END_IF;
ELSE
    DoorTimer1(IN := FALSE);
    DoorTimer2(IN := FALSE);
END_IF;

// === Check for Requests Above and Below ===
RequestsAbove := FALSE;
RequestsBelow := FALSE;

FOR i := CurrentFloor + 1 TO 5 DO
    IF (CabinRequest[i] OR (i <= 4 AND UpCall[i])) THEN
        RequestsAbove := TRUE;
    END_IF;
END_FOR;

FOR i := 1 TO CurrentFloor - 1 DO
    IF (CabinRequest[i] OR (i >= 2 AND DownCall[i])) THEN
        RequestsBelow := TRUE;
    END_IF;
END_FOR;

// === Direction and Movement Logic ===
IF DoorOpen THEN
    MoveUp := FALSE;
    MoveDown := FALSE;
ELSE
    IF GoingUp THEN
        IF RequestsAbove THEN
            MoveUp := TRUE;
            MoveDown := FALSE;
        ELSIF RequestsBelow THEN
            GoingUp := FALSE;
            MoveUp := FALSE;
            MoveDown := TRUE;
        ELSE
            Idle := TRUE;
            MoveUp := FALSE;
            MoveDown := FALSE;
        END_IF;
    ELSE
        IF RequestsBelow THEN
            MoveDown := TRUE;
            MoveUp := FALSE;
        ELSIF RequestsAbove THEN
            GoingUp := TRUE;
            MoveUp := TRUE;
            MoveDown := FALSE;
        ELSE
            Idle := TRUE;
            MoveUp := FALSE;
            MoveDown := FALSE;
        END_IF;
    END_IF;
END_IF;

// === Floor Tracking (Simulated) ===
// You'd replace this with actual position logic in a real implementation
IF MoveUp THEN
    IF CurrentFloor < 5 THEN CurrentFloor := CurrentFloor + 1; END_IF;
ELSIF MoveDown THEN
    IF CurrentFloor > 1 THEN CurrentFloor := CurrentFloor - 1; END_IF;
END_IF;
