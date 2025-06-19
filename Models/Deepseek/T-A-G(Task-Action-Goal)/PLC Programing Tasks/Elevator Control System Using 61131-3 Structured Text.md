PROGRAM ElevatorControl
VAR_INPUT
    AtFloor : ARRAY[1..5] OF BOOL; // Floor detection sensors
    CabinRequest : ARRAY[1..5] OF BOOL; // Buttons inside the elevator
    UpCall : ARRAY[1..4] OF BOOL; // Up call buttons on floors 1 to 4
    DownCall : ARRAY[2..5] OF BOOL; // Down call buttons on floors 2 to 5
    TopLimit : BOOL; // Optional top limit switch
    BottomLimit : BOOL; // Optional bottom limit switch
END_VAR

VAR_OUTPUT
    DoorOpen : BOOL; // Controls the elevator door
    MotorUp : BOOL; // Controls the motor to move up
    MotorDown : BOOL; // Controls the motor to move down
END_VAR

VAR
    CurrentFloor : INT := 1; // Current floor position
    GoingUp : BOOL := FALSE; // Direction flags
    GoingDown : BOOL := FALSE;
    Timer7s : TON; // 7-second timer for initial door open
    Timer10s : TON; // 10-second timer for extended door open
    AnyCabinRequest : BOOL; // Checks if there are any cabin requests
    AnyUpCall : BOOL; // Checks if there are any up calls
    AnyDownCall : BOOL; // Checks if there are any down calls
END_VAR

// Helper functions to check for any requests
FUNCTION CheckAnyCabinRequest : BOOL
BEGIN
    FOR i := 1 TO 5 DO
        IF CabinRequest[i] THEN
            RETURN TRUE;
        END_IF;
    END_FOR;
    RETURN FALSE;
END_FUNCTION

FUNCTION CheckAnyUpCall : BOOL
BEGIN
    FOR i := 1 TO 4 DO
        IF UpCall[i] THEN
            RETURN TRUE;
        END_IF;
    END_FOR;
    RETURN FALSE;
END_FUNCTION

FUNCTION CheckAnyDownCall : BOOL
BEGIN
    FOR i := 2 TO 5 DO
        IF DownCall[i] THEN
            RETURN TRUE;
        END_IF;
    END_FOR;
    RETURN FALSE;
END_FUNCTION

// Main execution loop
FOR i := 1 TO 5 DO
    IF AtFloor[i] THEN
        CurrentFloor := i;
        DoorOpen := TRUE;
        Timer7s(IN := TRUE, PT := T#7s);
        
        IF Timer7s.Q THEN
            AnyCabinRequest := CheckAnyCabinRequest();
            IF NOT AnyCabinRequest THEN
                Timer10s(IN := TRUE, PT := T#10s);
                IF Timer10s.Q THEN
                    DoorOpen := FALSE;
                END_IF;
            ELSE
                DoorOpen := FALSE;
            END_IF;
        END_IF;
    END_IF;
END_FOR;

// Reset timers when door closes
IF NOT DoorOpen THEN
    Timer7s(IN := FALSE);
    Timer10s(IN := FALSE);
END_IF;

// Determine direction based on current floor and requests
IF NOT DoorOpen THEN
    IF GoingUp THEN
        IF CurrentFloor < 5 THEN
            IF CabinRequest[CurrentFloor + 1] OR UpCall[CurrentFloor] THEN
                MotorUp := TRUE;
                MotorDown := FALSE;
            ELSE
                GoingUp := FALSE;
                GoingDown := TRUE;
            END_IF;
        ELSE
            GoingUp := FALSE;
            GoingDown := TRUE;
        END_IF;
    ELSIF GoingDown THEN
        IF CurrentFloor > 1 THEN
            IF CabinRequest[CurrentFloor - 1] OR DownCall[CurrentFloor] THEN
                MotorUp := FALSE;
                MotorDown := TRUE;
            ELSE
                GoingUp := TRUE;
                GoingDown := FALSE;
            END_IF;
        ELSE
            GoingUp := TRUE;
            GoingDown := FALSE;
        END_IF;
    ELSE
        // Determine initial direction based on first request
        AnyUpCall := CheckAnyUpCall();
        AnyDownCall := CheckAnyDownCall();

        IF AnyUpCall THEN
            GoingUp := TRUE;
            GoingDown := FALSE;
        ELSIF AnyDownCall THEN
            GoingUp := FALSE;
            GoingDown := TRUE;
        ELSE
            MotorUp := FALSE;
            MotorDown := FALSE;
        END_IF;
    END_IF;
ELSE
    MotorUp := FALSE;
    MotorDown := FALSE;
END_IF;

// Additional comments for clarity
// - When the elevator arrives at a floor (AtFloor[CurrentFloor] = TRUE):
//   - Open the door and run a 7-second timer.
//   - If no cabin request is received during this time, extend door opening by 10 more seconds using a second timer.
//   - After timeout, close the door automatically.
// - Direction and Movement Logic:
//   - Maintain a GoingUp or GoingDown flag.
//   - While moving, prioritize handling requests in the current direction.
//   - If no requests remain in the current direction, reverse direction.
//   - Elevator must not move if the door is open, ensuring passenger safety.



