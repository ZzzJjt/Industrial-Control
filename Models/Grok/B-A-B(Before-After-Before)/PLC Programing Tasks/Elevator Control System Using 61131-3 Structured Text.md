(* Elevator Control Logic for 5-Floor System in IEC 61131-3 Structured Text *)
(* Purpose: Manage elevator movement, door timing, and call prioritization with safety *)

PROGRAM ElevatorControl
VAR
    (* Inputs *)
    UpCall : ARRAY[1..4] OF BOOL;           (* Up call buttons on floors 1-4 *)
    DownCall : ARRAY[2..5] OF BOOL;         (* Down call buttons on floors 2-5 *)
    CabinRequest : ARRAY[1..5] OF BOOL;     (* Cabin floor selection buttons *)
    TopLimit : ARRAY[1..5] OF BOOL;         (* Top limit switch for each floor *)
    BottomLimit : ARRAY[1..5] OF BOOL;      (* Bottom limit switch for each floor *)
    AtFloor : ARRAY[1..5] OF BOOL;          (* Elevator positioned at floor *)

    (* Outputs *)
    MotorUp : BOOL;                         (* Command to move elevator up *)
    MotorDown : BOOL;                       (* Command to move elevator down *)
    DoorOpen : BOOL;                        (* Command to open/close door *)

    (* Internal Variables *)
    CurrentFloor : INT := 1;                (* Current elevator floor *)
    GoingUp : BOOL;                         (* TRUE if moving up *)
    GoingDown : BOOL;                       (* TRUE if moving down *)
    Idle : BOOL := TRUE;                    (* TRUE when no movement *)
    DoorTimer1 : TON;                       (* 7-second door open timer *)
    DoorTimer2 : TON;                       (* 10-second extended door timer *)
    AnyCabinButtonPressed : BOOL;           (* TRUE if any cabin button pressed *)
    TargetFloor : INT;                      (* Next floor to service *)
    HasPendingCalls : BOOL;                 (* TRUE if any calls are pending *)

END_VAR

(* Helper Function: Check for Pending Calls *)
FUNCTION CheckPendingCalls : BOOL
VAR
    i : INT;
END_VAR
CheckPendingCalls := FALSE;
FOR i := 1 TO 5 DO
    IF CabinRequest[i] THEN
        CheckPendingCalls := TRUE;
    END_IF;
END_FOR;
FOR i := 1 TO 4 DO
    IF UpCall[i] THEN
        CheckPendingCalls := TRUE;
    END_IF;
END_FOR;
FOR i := 2 TO 5 DO
    IF DownCall[i] THEN
        CheckPendingCalls := TRUE;
    END_IF;
END_FOR;
END_FUNCTION

(* Main Control Logic *)
(* 1. Update Current Floor based on Position Sensors *)
FOR i := 1 TO 5 DO
    IF AtFloor[i] AND (TopLimit[i] OR BottomLimit[i]) THEN
        CurrentFloor := i;
    END_IF;
END_FOR;

(* 2. Check Cabin Button Press *)
AnyCabinButtonPressed := FALSE;
FOR i := 1 TO 5 DO
    IF CabinRequest[i] THEN
        AnyCabinButtonPressed := TRUE;
    END_IF;
END_FOR;

(* 3. Door Control Logic *)
IF AtFloor[CurrentFloor] THEN
    Idle := TRUE;
    MotorUp := FALSE;
    MotorDown := FALSE;
    DoorOpen := TRUE;
    DoorTimer1(IN := TRUE, PT := T#7s);  (* Start 7-second door timer *)
    IF DoorTimer1.Q THEN
        IF NOT AnyCabinButtonPressed THEN
            DoorTimer2(IN := TRUE, PT := T#10s);  (* Extend door open for 10s *)
            IF DoorTimer2.Q THEN
                DoorOpen := FALSE;  (* Close door after extended time *)
                DoorTimer1(IN := FALSE);
                DoorTimer2(IN := FALSE);
            END_IF;
        ELSE
            DoorOpen := FALSE;  (* Close door if cabin button pressed *)
            DoorTimer1(IN := FALSE);
            DoorTimer2(IN := FALSE);
        END_IF;
    END_IF;
ELSE
    DoorTimer1(IN := FALSE);  (* Reset timers when not at floor *)
    DoorTimer2(IN := FALSE);
    DoorOpen := FALSE;
END_IF;

(* 4. Movement and Direction Logic *)
IF NOT DoorOpen THEN
    HasPendingCalls := CheckPendingCalls();
    IF HasPendingCalls THEN
        (* Find Target Floor in Current Direction *)
        TargetFloor := 0;
        IF GoingUp OR (NOT GoingDown AND CurrentFloor < 5) THEN
            (* Prioritize upward calls *)
            FOR i := CurrentFloor + 1 TO 5 DO
                IF CabinRequest[i] OR DownCall[i] OR (i < 5 AND UpCall[i]) THEN
                    TargetFloor := i;
                    EXIT;
                END_IF;
            END_FOR;
            IF TargetFloor = 0 THEN
                (* No upward calls, check downward *)
                GoingUp := FALSE;
                GoingDown := TRUE;
                FOR i := CurrentFloor - 1 DOWNTO 1 DO
                    IF CabinRequest[i] OR UpCall[i] OR (i > 1 AND DownCall[i]) THEN
                        TargetFloor := i;
                        EXIT;
                    END_IF;
                END_FOR;
            END_IF;
        ELSIF GoingDown OR (NOT GoingUp AND CurrentFloor > 1) THEN
            (* Prioritize downward calls *)
            FOR i := CurrentFloor - 1 DOWNTO 1 DO
                IF CabinRequest[i] OR UpCall[i] OR (i > 1 AND DownCall[i]) THEN
                    TargetFloor := i;
                    EXIT;
                END_IF;
            END_FOR;
            IF TargetFloor = 0 THEN
                (* No downward calls, check upward *)
                GoingDown := FALSE;
                GoingUp := TRUE;
                FOR i := CurrentFloor + 1 TO 5 DO
                    IF CabinRequest[i] OR DownCall[i] OR (i < 5 AND UpCall[i]) THEN
                        TargetFloor := i;
                        EXIT;
                    END_IF;
                END_FOR;
            END_IF;
        END_IF;

        (* Move to Target Floor *)
        IF TargetFloor > CurrentFloor THEN
            MotorUp := TRUE;
            MotorDown := FALSE;
            GoingUp := TRUE;
            GoingDown := FALSE;
            Idle := FALSE;
        ELSIF TargetFloor < CurrentFloor AND TargetFloor > 0 THEN
            MotorUp := FALSE;
            MotorDown := TRUE;
            GoingUp := FALSE;
            GoingDown := TRUE;
            Idle := FALSE;
        ELSE
            MotorUp := FALSE;
            MotorDown := FALSE;
            Idle := TRUE;
        END_IF;
    ELSE
        (* No pending calls, remain idle *)
        MotorUp := FALSE;
        MotorDown := FALSE;
        GoingUp := FALSE;
        GoingDown := FALSE;
        Idle := TRUE;
    END_IF;
END_IF;

(* Notes:
   - Safety: No movement allowed while DoorOpen is TRUE
   - Position Detection: AtFloor combined with TopLimit/BottomLimit confirms CurrentFloor
   - Door Timing: 7s initial open, extended by 10s if no cabin buttons pressed
   - Call Prioritization: Serves calls in current direction before reversing
   - Idle State: Elevator stops and waits when no pending calls
   - Physical Integration:
     - Inputs: Push-buttons for UpCall, DownCall, CabinRequest; limit switches for AtFloor, TopLimit, BottomLimit
     - Outputs: Relays for MotorUp, MotorDown, DoorOpen
     - Sensors: Ensure TopLimit/BottomLimit are calibrated to avoid false positives
   - Scalability: Extend arrays for additional floors
   - Maintenance: Add HMI to display CurrentFloor, direction, and pending calls
*)
END_PROGRAM
