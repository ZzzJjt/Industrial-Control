(* Program: 5-Floor Elevator Control *)
(* Version: 1.0, Date: 2025-05-21 *)
(* Manages floor detection, door operation, movement, and button requests *)
PROGRAM PRG_ElevatorControl
VAR
    (* Inputs *)
    AtFloor : ARRAY[1..5] OF BOOL;    (* TRUE if elevator is at floor i *)
    CabinRequest : ARRAY[1..5] OF BOOL; (* TRUE if cabin button for floor i pressed *)
    UpCall : ARRAY[1..4] OF BOOL;     (* TRUE if up call button at floor i pressed *)
    DownCall : ARRAY[2..5] OF BOOL;   (* TRUE if down call button at floor i pressed *)
    TopLimit : BOOL;                  (* TRUE if at top limit switch *)
    BottomLimit : BOOL;               (* TRUE if at bottom limit switch *)
    EmergencyStop : BOOL;             (* TRUE if emergency stop activated *)
    
    (* Outputs *)
    DoorOpen : BOOL;                  (* TRUE to open door *)
    MoveUp : BOOL;                    (* TRUE to move elevator up *)
    MoveDown : BOOL;                  (* TRUE to move elevator down *)
    AlarmActive : BOOL;               (* TRUE if emergency stop or error *)
    
    (* Internal Variables *)
    State : UINT;                     (* State: 0=Idle, 1=DoorOpening, 2=DoorOpen, 3=DoorClosing, 4=Moving *)
    CurrentFloor : UINT := 1;         (* Current floor, 1 to 5 *)
    TargetFloor : UINT;               (* Target floor for movement *)
    GoingUp : BOOL;                   (* TRUE if moving up *)
    GoingDown : BOOL;                 (* TRUE if moving down *)
    DoorTimer7s : TON;                (* 7-second door open timer *)
    DoorTimer10s : TON;               (* 10-second extended door timer *)
    AnyRequest : BOOL;                (* TRUE if any cabin or call request *)
    AnyRequestAbove : BOOL;           (* TRUE if requests above current floor *)
    AnyRequestBelow : BOOL;           (* TRUE if requests below current floor *)
    i : UINT;                         (* Loop index *)
END_VAR

(* Initialize outputs and state *)
DoorOpen := FALSE;
MoveUp := FALSE;
MoveDown := FALSE;
AlarmActive := FALSE;
State := 0;                           (* Start in Idle *)
DoorTimer7s(IN := FALSE, PT := T#7s); (* 7-second door timer *)
DoorTimer10s(IN := FALSE, PT := T#10s); (* 10-second extended timer *)

(* Main logic *)
(* Emergency stop handling: Overrides all operations *)
IF EmergencyStop THEN
    (* Halt all operations and reset to safe state *)
    DoorOpen := FALSE;                (* Close door *)
    MoveUp := FALSE;                  (* Stop upward movement *)
    MoveDown := FALSE;                (* Stop downward movement *)
    AlarmActive := TRUE;              (* Activate alarm *)
    DoorTimer7s(IN := FALSE);         (* Reset 7s timer *)
    DoorTimer10s(IN := FALSE);        (* Reset 10s timer *)
    State := 0;                       (* Return to Idle *)
ELSE
    (* Check floor detection and limit switches *)
    CurrentFloor := 0;
    FOR i := 1 TO 5 DO
        IF AtFloor[i] THEN
            CurrentFloor := i;
            EXIT;
        END_IF;
    END_FOR;
    
    (* Validate floor and limits *)
    IF CurrentFloor = 0 OR (TopLimit AND CurrentFloor <> 5) OR (BottomLimit AND CurrentFloor <> 1) THEN
        (* Invalid floor detection or limit switch mismatch *)
        DoorOpen := FALSE;
        MoveUp := FALSE;
        MoveDown := FALSE;
        AlarmActive := TRUE;
        State := 0;
        RETURN;
    END_IF;
    
    (* Check for any requests *)
    AnyRequest := FALSE;
    AnyRequestAbove := FALSE;
    AnyRequestBelow := FALSE;
    FOR i := 1 TO 5 DO
        IF CabinRequest[i] THEN
            AnyRequest := TRUE;
            IF i > CurrentFloor THEN
                AnyRequestAbove := TRUE;
            ELSIF i < CurrentFloor THEN
                AnyRequestBelow := TRUE;
            END_IF;
        END_IF;
    END_FOR;
    FOR i := 1 TO 4 DO
        IF UpCall[i] THEN
            AnyRequest := TRUE;
            IF i >= CurrentFloor THEN
                AnyRequestAbove := TRUE;
            END_IF;
        END_IF;
    END_FOR;
    FOR i := 2 TO 5 DO
        IF DownCall[i] THEN
            AnyRequest := TRUE;
            IF i <= CurrentFloor THEN
                AnyRequestBelow := TRUE;
            END_IF;
        END_IF;
    END_FOR;
    
    (* State machine *)
    CASE State OF
        0: (* Idle *)
            (* Wait for requests and door closed *)
            IF NOT DoorOpen AND AnyRequest THEN
                (* Check if current floor is requested *)
                IF CabinRequest[CurrentFloor] OR 
                   (UpCall[CurrentFloor] AND CurrentFloor <= 4) OR 
                   (DownCall[CurrentFloor] AND CurrentFloor >= 2) THEN
                    State := 1;           (* Move to DoorOpening *)
                ELSE
                    (* Find target floor *)
                    IF GoingUp AND AnyRequestAbove THEN
                        FOR i := CurrentFloor + 1 TO 5 DO
                            IF CabinRequest[i] OR (DownCall[i] AND i >= 2) OR 
                               (UpCall[i] AND i <= 4) THEN
                                TargetFloor := i;
                                MoveUp := TRUE;
                                MoveDown := FALSE;
                                State := 4;   (* Move to Moving *)
                                EXIT;
                            END_IF;
                        END_FOR;
                    ELSIF GoingDown AND AnyRequestBelow THEN
                        FOR i := CurrentFloor - 1 DOWNTO 1 DO
                            IF CabinRequest[i] OR (UpCall[i] AND i <= 4) OR 
                               (DownCall[i] AND i >= 2) THEN
                                TargetFloor := i;
                                MoveUp := FALSE;
                                MoveDown := TRUE;
                                State := 4;   (* Move to Moving *)
                                EXIT;
                            END_IF;
                        END_FOR;
                    ELSE
                        (* Reverse direction if no requests in current direction *)
                        IF AnyRequestAbove THEN
                            GoingUp := TRUE;
                            GoingDown := FALSE;
                        ELSIF AnyRequestBelow THEN
                            GoingUp := FALSE;
                            GoingDown := TRUE;
                        END_IF;
                    END_IF;
                END_IF;
            END_IF;

        1: (* DoorOpening *)
            (* Open door and start 7s timer *)
            DoorOpen := TRUE;
            DoorTimer7s(IN := TRUE);
            State := 2;                   (* Move to DoorOpen *)

        2: (* DoorOpen *)
            (* Monitor timers and requests *)
            IF DoorTimer7s.Q THEN
                (* 7s elapsed: Check for cabin requests *)
                IF NOT CabinRequest[CurrentFloor] THEN
                    DoorTimer10s(IN := TRUE); (* Start 10s extended timer *)
                    IF DoorTimer10s.Q THEN
                        (* 10s elapsed: Close door *)
                        DoorOpen := FALSE;
                        DoorTimer7s(IN := FALSE);
                        DoorTimer10s(IN := FALSE);
                        State := 3;           (* Move to DoorClosing *)
                    END_IF;
                ELSE
                    (* Cabin request: Close door immediately *)
                    DoorOpen := FALSE;
                    DoorTimer7s(IN := FALSE);
                    DoorTimer10s(IN := FALSE);
                    State := 3;           (* Move to DoorClosing *)
                END_IF;
            END_IF;

        3: (* DoorClosing *)
            (* Wait for door to fully close *)
            (* Assume door closes instantly for simplicity *)
            IF NOT DoorOpen THEN
                (* Clear requests for current floor *)
                CabinRequest[CurrentFloor] := FALSE;
                IF CurrentFloor <= 4 THEN
                    UpCall[CurrentFloor] := FALSE;
                END_IF;
                IF CurrentFloor >= 2 THEN
                    DownCall[CurrentFloor] := FALSE;
                END_IF;
                State := 0;               (* Return to Idle *)
            END_IF;

        4: (* Moving *)
            (* Continue moving until target floor reached *)
            IF AtFloor[TargetFloor] THEN
                MoveUp := FALSE;
                MoveDown := FALSE;
                State := 1;               (* Move to DoorOpening *)
            ELSIF TopLimit AND MoveUp THEN
                MoveUp := FALSE;
                GoingUp := FALSE;
                GoingDown := TRUE;
                State := 0;               (* Return to Idle *)
            ELSIF BottomLimit AND MoveDown THEN
                MoveDown := FALSE;
                GoingUp := TRUE;
                GoingDown := FALSE;
                State := 0;               (* Return to Idle *)
            END_IF;
    END_CASE;
END_IF;

(* Ensure no movement when door is open *)
IF DoorOpen THEN
    MoveUp := FALSE;
    MoveDown := FALSE;
END_IF;

END_PROGRAM
