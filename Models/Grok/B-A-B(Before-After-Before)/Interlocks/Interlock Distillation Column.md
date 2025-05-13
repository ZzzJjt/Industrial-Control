(* Program: ELEVATOR_CONTROL
   Purpose: Controls a 5-floor elevator system with position detection, door timing, and call request handling.
   Features:
   - Detects elevator position using top and bottom limit switches
   - Manages door open/close with 7s initial open and 10s reopen if no cabin request
   - Prioritizes floor calls and cabin requests in current direction
   - Ensures safety with door-open motion prevention and idle state
*)

PROGRAM ELEVATOR_CONTROL
VAR
    (* Inputs *)
    UpCall : ARRAY[1..4] OF BOOL;        (* Up call buttons for floors 1-4 *)
    DownCall : ARRAY[2..5] OF BOOL;      (* Down call buttons for floors 2-5 *)
    CabinRequest : ARRAY[1..5] OF BOOL;  (* Cabin floor selection buttons *)
    TopLimit : ARRAY[1..5] OF BOOL;      (* Top limit switch for each floor *)
    BottomLimit : ARRAY[1..5] OF BOOL;   (* Bottom limit switch for each floor *)
    
    (* Outputs *)
    MoveUp : BOOL;                       (* Command to move elevator up *)
    MoveDown : BOOL;                     (* Command to move elevator down *)
    DoorOpen : BOOL;                     (* Command to open/close door *)
    CurrentFloor : UINT := 1;            (* Current floor, 1-5 *)
    Idle : BOOL := TRUE;                 (* TRUE when elevator is idle *)
    
    (* Internal variables *)
    GoingUp : BOOL;                      (* TRUE if moving up, FALSE if down *)
    DoorTimer1 : TON;                    (* 7s timer for initial door open *)
    DoorTimer2 : TON;                    (* 10s timer for reopen *)
    AnyCabinButtonPressed : BOOL;         (* TRUE if any cabin button is pressed *)
    TargetFloor : UINT;                  (* Next floor to service *)
    HasPendingCalls : BOOL;              (* TRUE if any calls are pending *)
    i : UINT;                            (* Loop index *)
END_VAR

(* Initialize outputs *)
MoveUp := FALSE;
MoveDown := FALSE;
DoorOpen := FALSE;
Idle := TRUE;

(* Main logic *)
(* Step 1: Determine current floor based on limit switches *)
FOR i := 1 TO 5 DO
    IF TopLimit[i] AND BottomLimit[i] THEN
        CurrentFloor := i;
        EXIT;
    END_IF
END_FOR

(* Step 2: Check for pending calls *)
HasPendingCalls := FALSE;
FOR i := 1 TO 4 DO
    IF UpCall[i] THEN HasPendingCalls := TRUE; END_IF
END_FOR
FOR i := 2 TO 5 DO
    IF DownCall[i] THEN HasPendingCalls := TRUE; END_IF
END_FOR
FOR i := 1 TO 5 DO
    IF CabinRequest[i] THEN HasPendingCalls := TRUE; END_IF
END_FOR

(* Step 3: Door control and state management *)
IF TopLimit[CurrentFloor] AND BottomLimit[CurrentFloor] THEN
    (* Elevator is at a floor *)
    Idle := FALSE;
    MoveUp := FALSE;
    MoveDown := FALSE;
    
    (* Open door and start initial timer *)
    DoorOpen := TRUE;
    DoorTimer1(IN := TRUE, PT := T#7s);
    
    (* Check if any cabin button is pressed *)
    AnyCabinButtonPressed := FALSE;
    FOR i := 1 TO 5 DO
        IF CabinRequest[i] THEN
            AnyCabinButtonPressed := TRUE;
            EXIT;
        END_IF
    END_FOR
    
    (* Handle door timing *)
    IF DoorTimer1.Q THEN
        IF NOT AnyCabinButtonPressed THEN
            (* No cabin button pressed, reopen for 10s *)
            DoorTimer2(IN := TRUE, PT := T#10s);
            IF DoorTimer2.Q THEN
                DoorOpen := FALSE;
                DoorTimer1(IN := FALSE);
                DoorTimer2(IN := FALSE);
            END_IF
        ELSE
            (* Cabin button pressed, close door *)
            DoorOpen := FALSE;
            DoorTimer1(IN := FALSE);
            DoorTimer2(IN := FALSE);
        END_IF
    END_IF
ELSE
    (* Elevator is between floors *)
    DoorOpen := FALSE;
    DoorTimer1(IN := FALSE);
    DoorTimer2(IN := FALSE);
END_IF

(* Step 4: Movement and direction logic *)
IF NOT DoorOpen AND HasPendingCalls THEN
    (* Determine target floor *)
    TargetFloor := 0;
    IF GoingUp THEN
        (* Service calls above current floor *)
        FOR i := CurrentFloor + 1 TO 5 DO
            IF CabinRequest[i] OR (i < 5 AND UpCall[i]) OR DownCall[i] THEN
                TargetFloor := i;
                EXIT;
            END_IF
        END_FOR
        IF TargetFloor = 0 THEN
            (* No calls above, check below and reverse direction *)
            FOR i := CurrentFloor - 1 DOWNTO 1 DO
                IF CabinRequest[i] OR UpCall[i] OR (i > 1 AND DownCall[i]) THEN
                    TargetFloor := i;
                    GoingUp := FALSE;
                    EXIT;
                END_IF
            END_FOR
        END_IF
    ELSE
        (* Service calls below current floor *)
        FOR i := CurrentFloor - 1 DOWNTO 1 DO
            IF CabinRequest[i] OR UpCall[i] OR (i > 1 AND DownCall[i]) THEN
                TargetFloor := i;
                EXIT;
            END_IF
        END_FOR
        IF TargetFloor = 0 THEN
            (* No calls below, check above and reverse direction *)
            FOR i := CurrentFloor + 1 TO 5 DO
                IF CabinRequest[i] OR (i < 5 AND UpCall[i]) OR DownCall[i] THEN
                    TargetFloor := i;
                    GoingUp := TRUE;
                    EXIT;
                END_IF
            END_FOR
        END_IF
    END_IF
    
    (* Move to target floor *)
    IF TargetFloor > 0 THEN
        IF TargetFloor > CurrentFloor THEN
            MoveUp := TRUE;
            MoveDown := FALSE;
            GoingUp := TRUE;
        ELSIF TargetFloor < CurrentFloor THEN
            MoveUp := FALSE;
            MoveDown := TRUE;
            GoingUp := FALSE;
        END_IF
    END_IF
ELSE
    (* No pending calls or door open *)
    MoveUp := FALSE;
    MoveDown := FALSE;
    Idle := NOT DoorOpen AND NOT HasPendingCalls;
END_IF

(* Step 5: Clear served requests *)
IF TopLimit[CurrentFloor] AND BottomLimit[CurrentFloor] THEN
    CabinRequest[CurrentFloor] := FALSE;
    IF CurrentFloor < 5 THEN
        UpCall[CurrentFloor] := FALSE;
    END_IF
    IF CurrentFloor > 1 THEN
        DownCall[CurrentFloor] := FALSE;
    END_IF
END_IF

END_PROGRAM
