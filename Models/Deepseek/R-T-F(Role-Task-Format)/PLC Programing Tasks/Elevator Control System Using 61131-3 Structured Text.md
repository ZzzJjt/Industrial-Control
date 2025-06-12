PROGRAM ElevatorControl
VAR
    AtFloor : ARRAY[1..5] OF BOOL; // Limit switches indicating elevator position at each floor
    UpCall : ARRAY[1..4] OF BOOL;   // Call buttons for moving up (floors 1 to 4)
    DownCall : ARRAY[2..5] OF BOOL; // Call buttons for moving down (floors 2 to 5)
    CabinRequest : ARRAY[1..5] OF BOOL; // Cabin request buttons for selecting destination floors
    GoingUp, GoingDown : BOOL;      // Direction indicators
    CurrentFloor : INT := 1;         // Current floor number (initially ground floor)
    DoorOpen : BOOL;                 // Indicates if the door is open
    Timer7s : TON;                   // Timer for keeping the door open for 7 seconds
    Timer10s : TON;                  // Timer for extending the door open time by 10 seconds
    AnyCabinRequest : BOOL;           // Flag indicating any cabin request is active
    AnyUpCall : BOOL;                // Flag indicating any up call is active
    AnyDownCall : BOOL;              // Flag indicating any down call is active
END_VAR

// Determine if there are any cabin requests
AnyCabinRequest := CabinRequest[CurrentFloor];

// Determine if there are any up calls
AnyUpCall := UpCall[1] OR UpCall[2] OR UpCall[3] OR UpCall[4];

// Determine if there are any down calls
AnyDownCall := DownCall[2] OR DownCall[3] OR DownCall[4] OR DownCall[5];

// Floor Detection and Door Control Logic
IF AtFloor[CurrentFloor] THEN
    DoorOpen := TRUE;
    Timer7s(IN := TRUE, PT := T#7s);

    IF Timer7s.Q THEN
        IF NOT AnyCabinRequest THEN
            Timer10s(IN := TRUE, PT := T#10s);
            IF Timer10s.Q THEN
                DoorOpen := FALSE;
                Timer7s(IN := FALSE); // Reset timer
                Timer10s(IN := FALSE); // Reset timer
            END_IF;
        ELSE
            DoorOpen := FALSE;
            Timer7s(IN := FALSE); // Reset timer
            Timer10s(IN := FALSE); // Reset timer
        END_IF;
    END_IF;
ELSE
    DoorOpen := FALSE;
    Timer7s(IN := FALSE); // Reset timer
    Timer10s(IN := FALSE); // Reset timer
END_IF;

// Movement and Direction Logic
IF NOT DoorOpen THEN
    IF GoingUp THEN
        // Check for requests above the current floor
        FOR i := CurrentFloor + 1 TO 5 DO
            IF CabinRequest[i] OR UpCall[i - 1] THEN
                CurrentFloor := i;
                EXIT;
            END_IF;
        END_FOR;

        // If no more requests above, check for down calls
        IF CurrentFloor = 5 THEN
            GoingUp := FALSE;
            GoingDown := TRUE;
        END_IF;
    ELSIF GoingDown THEN
        // Check for requests below the current floor
        FOR i := CurrentFloor - 1 DOWNTO 1 BY -1 DO
            IF CabinRequest[i] OR DownCall[i + 1] THEN
                CurrentFloor := i;
                EXIT;
            END_IF;
        END_FOR;

        // If no more requests below, check for up calls
        IF CurrentFloor = 1 THEN
            GoingUp := TRUE;
            GoingDown := FALSE;
        END_IF;
    END_IF;
END_IF;

// Initial direction setting based on first request
IF NOT GoingUp AND NOT GoingDown THEN
    IF AnyUpCall THEN
        GoingUp := TRUE;
    ELSIF AnyDownCall THEN
        GoingDown := TRUE;
    END_IF;
END_IF;

// Ignore repeated or invalid requests if the elevator is already at the requested floor
FOR i := 1 TO 5 DO
    IF CabinRequest[i] AND i = CurrentFloor THEN
        CabinRequest[i] := FALSE;
    END_IF;
END_FOR;

FOR i := 1 TO 4 DO
    IF UpCall[i] AND i + 1 = CurrentFloor THEN
        UpCall[i] := FALSE;
    END_IF;
END_FOR;

FOR i := 2 TO 5 DO
    IF DownCall[i] AND i = CurrentFloor THEN
        DownCall[i] := FALSE;
    END_IF;
END_FOR;
