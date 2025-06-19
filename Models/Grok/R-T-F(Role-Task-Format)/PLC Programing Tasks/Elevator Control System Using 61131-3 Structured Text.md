PROGRAM ElevatorControl
VAR
    (* Inputs *)
    AtFloor : ARRAY[1..5] OF BOOL; (* TRUE if elevator is at floor i *)
    UpCall : ARRAY[1..4] OF BOOL; (* TRUE if up call from floor i *)
    DownCall : ARRAY[2..5] OF BOOL; (* TRUE if down call from floor i *)
    CabinRequest : ARRAY[1..5] OF BOOL; (* TRUE if cabin requests floor i *)
    
    (* Outputs *)
    DoorOpen : BOOL; (* TRUE to open door *)
    GoingUp : BOOL; (* TRUE if moving up *)
    GoingDown : BOOL; (* TRUE if moving down *)
    CurrentFloor : INT := 1; (* Current floor, 1 to 5 *)
    
    (* Internal variables *)
    State : INT := 0; (* 0=Idle, 1=DoorOpen7s, 2=DoorOpen10s, 3=Moving *)
    Timer7s : TON; (* 7-second timer for initial door open *)
    Timer10s : TON; (* 10-second timer for extended door open *)
    AnyCabinRequest : BOOL; (* TRUE if any cabin request is active *)
    AnyRequestAbove : BOOL; (* TRUE if requests exist above current floor *)
    AnyRequestBelow : BOOL; (* TRUE if requests exist below current floor *)
    i : INT; (* Loop index *)
    TargetFloor : INT; (* Next floor to move to *)
END_VAR

(* Initialize outputs *)
DoorOpen := FALSE;
GoingUp := FALSE;
GoingDown := FALSE;

(* Detect current floor *)
FOR i := 1 TO 5 DO
    IF AtFloor[i] THEN
        CurrentFloor := i;
        EXIT;
    END_IF;
END_FOR;

(* Check for cabin requests *)
AnyCabinRequest := FALSE;
FOR i := 1 TO 5 DO
    IF CabinRequest[i] AND i <> CurrentFloor THEN
        AnyCabinRequest := TRUE;
        EXIT;
    END_IF;
END_FOR;

(* Check for requests above/below current floor *)
AnyRequestAbove := FALSE;
AnyRequestBelow := FALSE;
FOR i := 1 TO 5 DO
    IF i > CurrentFloor THEN
        IF CabinRequest[i] OR (i <= 4 AND UpCall[i]) OR (i >= 2 AND DownCall[i]) THEN
            AnyRequestAbove := TRUE;
        END_IF;
    ELSIF i < CurrentFloor THEN
        IF CabinRequest[i] OR (i <= 4 AND UpCall[i]) OR (i >= 2 AND DownCall[i]) THEN
            AnyRequestBelow := TRUE;
        END_IF;
    END_IF;
END_FOR;

(* State machine *)
CASE State OF
    0: (* Idle *)
        (* At a floor, no door open *)
        IF AtFloor[CurrentFloor] THEN
            (* Open door and start 7-second timer *)
            DoorOpen := TRUE;
            Timer7s(IN := TRUE, PT := T#7s);
            State := 1;
        END_IF;

    1: (* Door open for 7 seconds *)
        IF Timer7s.Q THEN
            IF NOT AnyCabinRequest THEN
                (* No cabin request: extend door open for 10 seconds *)
                Timer10s(IN := TRUE, PT := T#10s);
                State := 2;
            ELSE
                (* Cabin request: close door and prepare to move *)
                DoorOpen := FALSE;
                Timer7s(IN := FALSE);
                State := 3;
            END_IF;
        END_IF;

    2: (* Door open for additional 10 seconds *)
        IF Timer10s.Q THEN
            (* Close door and prepare to move *)
            DoorOpen := FALSE;
            Timer10s(IN := FALSE);
            State := 3;
        END_IF;

    3: (* Moving or preparing to move *)
        (* Ensure door is closed before moving *)
        IF NOT DoorOpen THEN
            (* Find next target floor *)
            TargetFloor := 0;
            IF GoingUp THEN
                (* Prioritize requests above in up direction *)
                FOR i := CurrentFloor + 1 TO 5 DO
                    IF CabinRequest[i] OR (i <= 4 AND UpCall[i]) OR (i >= 2 AND DownCall[i]) THEN
                        TargetFloor := i;
                        EXIT;
                    END_IF;
                END_FOR;
                IF TargetFloor = 0 AND AnyRequestBelow THEN
                    (* No requests above: reverse to down *)
                    GoingUp := FALSE;
                    GoingDown := TRUE;
                END_IF;
            ELSIF GoingDown THEN
                (* Prioritize requests below in down direction *)
                FOR i := CurrentFloor - 1 DOWNTO 1 DO
                    IF CabinRequest[i] OR (i <= 4 AND UpCall[i]) OR (i >= 2 AND DownCall[i]) THEN
                        TargetFloor := i;
                        EXIT;
                    END_IF;
                END_FOR;
                IF TargetFloor = 0 AND AnyRequestAbove THEN
                    (* No requests below: reverse to up *)
                    GoingUp := TRUE;
                    GoingDown := FALSE;
                END_IF;
            ELSE
                (* No direction: check any request *)
                IF AnyRequestAbove THEN
                    GoingUp := TRUE;
                    FOR i := CurrentFloor + 1 TO 5 DO
                        IF CabinRequest[i] OR (i <= 4 AND UpCall[i]) OR (i >= 2 AND DownCall[i]) THEN
                            TargetFloor := i;
                            EXIT;
                        END_IF;
                    END_FOR;
                ELSIF AnyRequestBelow THEN
                    GoingDown := TRUE;
                    FOR i := CurrentFloor - 1 DOWNTO 1 DO
                        IF CabinRequest[i] OR (i <= 4 AND UpCall[i]) OR (i >= 2 AND DownCall[i]) THEN
                            TargetFloor := i;
                            EXIT;
                        END_IF;
                    END_FOR;
                END_IF;
            END_IF;

            (* Move if target found, else return to idle *)
            IF TargetFloor <> 0 THEN
                IF TargetFloor > CurrentFloor THEN
                    GoingUp := TRUE;
                    GoingDown := FALSE;
                ELSIF TargetFloor < CurrentFloor THEN
                    GoingUp := FALSE;
                    GoingDown := TRUE;
                END_IF;
            ELSE
                GoingUp := FALSE;
                GoingDown := FALSE;
                State := 0; (* No requests: return to idle *)
            END_IF;
        END_IF;

    ELSE
        (* Invalid state: reset *)
        DoorOpen := FALSE;
        GoingUp := FALSE;
        GoingDown := FALSE;
        Timer7s(IN := FALSE);
        Timer10s(IN := FALSE);
        State := 0;
END_CASE;

(* Safety: prevent motion if door is open *)
IF DoorOpen THEN
    GoingUp := FALSE;
    GoingDown := FALSE;
END_IF;

(* Reset timers if not at a floor *)
IF NOT AtFloor[CurrentFloor] THEN
    Timer7s(IN := FALSE);
    Timer10s(IN := FALSE);
END_IF;

END_PROGRAM
