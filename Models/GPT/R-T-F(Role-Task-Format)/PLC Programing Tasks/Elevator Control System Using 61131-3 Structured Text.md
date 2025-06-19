PROGRAM ElevatorControl
VAR
    // Floor Position Detection
    AtFloor : ARRAY[1..5] OF BOOL;
    CurrentFloor : INT := 1;

    // Request Buttons
    CabinRequest : ARRAY[1..5] OF BOOL;
    UpCall       : ARRAY[1..4] OF BOOL;
    DownCall     : ARRAY[2..5] OF BOOL;

    // Movement
    GoingUp   : BOOL := TRUE;
    GoingDown : BOOL := FALSE;
    MoveUp    : BOOL := FALSE;
    MoveDown  : BOOL := FALSE;

    // Door Logic
    DoorOpen   : BOOL := FALSE;
    DoorTimer7s : TON;
    DoorTimer10s : TON;
    AnyCabinRequest : BOOL := FALSE;

    // Internal Control
    i : INT;
END_VAR

// --- Update CurrentFloor Based on Floor Sensor ---
FOR i := 1 TO 5 DO
    IF AtFloor[i] THEN
        CurrentFloor := i;
    END_IF;
END_FOR

// --- Detect if any request is active (for door hold logic) ---
AnyCabinRequest := FALSE;
FOR i := 1 TO 5 DO
    IF CabinRequest[i] THEN
        AnyCabinRequest := TRUE;
    END_IF;
END_FOR

// --- Door Control Logic ---
IF AtFloor[CurrentFloor] THEN
    DoorOpen := TRUE;

    // Start 7-second door open timer
    DoorTimer7s(IN := TRUE, PT := T#7s);

    // After 7s, extend door open time if no request
    IF DoorTimer7s.Q THEN
        IF NOT AnyCabinRequest THEN
            DoorTimer10s(IN := TRUE, PT := T#10s);
            IF DoorTimer10s.Q THEN
                DoorOpen := FALSE;
                DoorTimer7s(IN := FALSE);
                DoorTimer10s(IN := FALSE);
            END_IF;
        ELSE
            DoorOpen := FALSE;
            DoorTimer7s(IN := FALSE);
            DoorTimer10s(IN := FALSE);
        END_IF;
    END_IF;
ELSE
    // Reset door timers if not at floor
    DoorTimer7s(IN := FALSE);
    DoorTimer10s(IN := FALSE);
END_IF

// --- Movement Logic (Only If Door is Closed) ---
IF NOT DoorOpen THEN
    MoveUp := FALSE;
    MoveDown := FALSE;

    // Going Up Logic
    IF GoingUp THEN
        FOR i := CurrentFloor+1 TO 5 DO
            IF UpCall[i-1] OR CabinRequest[i] THEN
                MoveUp := TRUE;
                EXIT;
            END_IF;
        END_FOR

        // If no more up requests, reverse
        IF NOT MoveUp THEN
            GoingUp := FALSE;
            GoingDown := TRUE;
        END_IF

    // Going Down Logic
    ELSIF GoingDown THEN
        FOR i := CurrentFloor-1 DOWNTO 1 DO
            IF DownCall[i+1] OR CabinRequest[i] THEN
                MoveDown := TRUE;
                EXIT;
            END_IF;
        END_FOR

        // If no more down requests, reverse
        IF NOT MoveDown THEN
            GoingDown := FALSE;
            GoingUp := TRUE;
        END_IF
    END_IF;
ELSE
    // Door is open — ensure no movement
    MoveUp := FALSE;
    MoveDown := FALSE;
END_IF
