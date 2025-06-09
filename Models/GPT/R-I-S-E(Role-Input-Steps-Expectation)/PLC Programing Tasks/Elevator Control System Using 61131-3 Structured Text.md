PROGRAM ElevatorControl
VAR_INPUT
    AtFloor      : ARRAY[1..5] OF BOOL;          // Floor sensors
    CabinRequest : ARRAY[1..5] OF BOOL;          // Internal cabin button
    UpCall       : ARRAY[1..4] OF BOOL;          // Up buttons (floors 1–4)
    DownCall     : ARRAY[2..5] OF BOOL;          // Down buttons (floors 2–5)
END_VAR

VAR_OUTPUT
    DoorOpen     : BOOL;                         // Door state
    MoveUp       : BOOL;                         // Motor up signal
    MoveDown     : BOOL;                         // Motor down signal
    CurrentFloor : INT := 1;                     // Elevator position index
    GoingUp      : BOOL := TRUE;                 // Direction flag
END_VAR

VAR
    i            : INT;
    AnyCabinReq  : BOOL;
    AnyUpReq     : BOOL;
    AnyDownReq   : BOOL;
    T_DoorOpen   : TON;                          // 7s door open timer
    T_ExtendOpen : TON;                          // 10s extended timer
    DoorTimerDone : BOOL := FALSE;
END_VAR

// === Detect Current Floor ===
FOR i := 1 TO 5 DO
    IF AtFloor[i] THEN
        CurrentFloor := i;
    END_IF;
END_FOR

// === Door Control Logic ===
IF AtFloor[CurrentFloor] THEN
    DoorOpen := TRUE;
    T_DoorOpen(IN := TRUE, PT := T#7s);
ELSE
    T_DoorOpen(IN := FALSE);
END_IF

T_DoorOpen(); // call timer

// Check if cabin requests are active
AnyCabinReq := FALSE;
FOR i := 1 TO 5 DO
    IF CabinRequest[i] THEN
        AnyCabinReq := TRUE;
    END_IF;
END_FOR

// Handle door open extensions
IF T_DoorOpen.Q THEN
    IF NOT AnyCabinReq THEN
        T_ExtendOpen(IN := TRUE, PT := T#10s);
        T_ExtendOpen();
        IF T_ExtendOpen.Q THEN
            DoorOpen := FALSE;
            T_ExtendOpen(IN := FALSE);
        END_IF;
    ELSE
        DoorOpen := FALSE;
        T_ExtendOpen(IN := FALSE);
    END_IF;
END_IF

// === Motion Blocking When Door Is Open ===
IF DoorOpen THEN
    MoveUp := FALSE;
    MoveDown := FALSE;
ELSE
    // === Determine Up/Down Requests ===
    AnyUpReq := FALSE;
    FOR i := CurrentFloor + 1 TO 5 DO
        IF CabinRequest[i] OR UpCall[i - 1] THEN
            AnyUpReq := TRUE;
        END_IF;
    END_FOR

    AnyDownReq := FALSE;
    FOR i := 1 TO CurrentFloor - 1 DO
        IF CabinRequest[i] OR DownCall[i] THEN
            AnyDownReq := TRUE;
        END_IF;
    END_FOR

    // === Movement Direction Logic ===
    IF GoingUp THEN
        IF AnyUpReq THEN
            MoveUp := TRUE;
            MoveDown := FALSE;
        ELSIF AnyDownReq THEN
            GoingUp := FALSE;
            MoveUp := FALSE;
            MoveDown := TRUE;
        ELSE
            MoveUp := FALSE;
            MoveDown := FALSE;
        END_IF
    ELSE
        IF AnyDownReq THEN
            MoveDown := TRUE;
            MoveUp := FALSE;
        ELSIF AnyUpReq THEN
            GoingUp := TRUE;
            MoveDown := FALSE;
            MoveUp := TRUE;
        ELSE
            MoveUp := FALSE;
            MoveDown := FALSE;
        END_IF
    END_IF
END_IF
