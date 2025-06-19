FUNCTION_BLOCK FB_ElevatorController
VAR_INPUT
    // Position Detection
    AtFloor : ARRAY [1..5] OF BOOL; // TRUE when elevator is at floor
    TopLimit, BottomLimit : BOOL;   // Safety limit switches

    // Cabin Buttons
    CabinRequest : ARRAY [1..5] OF BOOL;

    // Floor Call Buttons
    UpCall : ARRAY [1..4] OF BOOL;     // Up call from floor 1–4
    DownCall : ARRAY [2..5] OF BOOL;   // Down call from floor 2–5

    // Simulation Time Input
    CycleTime_ms : DINT; // Used for internal timers (e.g., from TON function block)
END_VAR

VAR_OUTPUT
    // Motion Control
    GoingUp : BOOL := FALSE;
    GoingDown : BOOL := FALSE;
    MotorRun : BOOL := FALSE;

    // Door Logic
    DoorOpen : BOOL := FALSE;
    DoorClosed : BOOL := TRUE;

    CurrentFloor : INT := 1;
    ElevatorState : STRING(20) := 'IDLE';
END_VAR

VAR
    // Internal state tracking
    State : INT := 0;
    Timer1 : TON; // 7s timer
    Timer2 : TON; // 10s timer
    DoorTimerStarted : BOOL := FALSE;
    AnyPendingRequest : BOOL := FALSE;
    TargetFloor : INT := 1;
END_VAR

// --- STEP 1: Determine Current Floor ---
FOR CurrentFloor := 1 TO 5 DO
    IF AtFloor[CurrentFloor] THEN
        EXIT;
    END_IF;
END_FOR;

// --- STEP 2: Check if any request exists (cabin or call) ---
AnyPendingRequest :=
    (CabinRequest[1] OR CabinRequest[2] OR CabinRequest[3] OR CabinRequest[4] OR CabinRequest[5]) OR
    (UpCall[1] OR UpCall[2] OR UpCall[3] OR UpCall[4]) OR
    (DownCall[2] OR DownCall[3] OR DownCall[4] OR DownCall[5]);

// --- STEP 3: Main State Machine ---
CASE State OF
    0: // IDLE - Wait for request
        ElevatorState := 'IDLE';
        IF AnyPendingRequest THEN
            State := 1;
        END_IF;

    1: // DECIDE DIRECTION
        ElevatorState := 'DECIDING_DIRECTION';

        // Decide target floor and direction
        IF GoingUp THEN
            FOR TargetFloor := CurrentFloor + 1 TO 5 DO
                IF CabinRequest[TargetFloor] OR (TargetFloor <= 4 AND UpCall[TargetFloor]) THEN
                    State := 2;
                    EXIT;
                END_IF;
            END_FOR;
            IF TargetFloor > CurrentFloor THEN
                GoingUp := TRUE;
                GoingDown := FALSE;
            ELSE
                GoingUp := FALSE;
                State := 3;
            END_IF;
        ELSIF GoingDown THEN
            FOR TargetFloor := CurrentFloor - 1 DOWNTO 1 DO
                IF CabinRequest[TargetFloor] OR (TargetFloor >= 2 AND DownCall[TargetFloor]) THEN
                    State := 2;
                    EXIT;
                END_IF;
            END_FOR;
            IF TargetFloor < CurrentFloor THEN
                GoingDown := TRUE;
                GoingUp := FALSE;
            ELSE
                GoingDown := FALSE;
                State := 3;
            END_IF;
        ELSE
            // No current direction, decide based on highest priority request
            FOR TargetFloor := 5 DOWNTO 1 DO
                IF CabinRequest[TargetFloor] THEN
                    IF TargetFloor > CurrentFloor THEN
                        GoingUp := TRUE;
                    ELSE
                        GoingDown := TRUE;
                    END_IF;
                    State := 2;
                    EXIT;
                END_IF;
            END_FOR;
        END_IF;

    2: // MOVING
        ElevatorState := 'MOVING';
        IF DoorOpen THEN
            MotorRun := FALSE;
            ElevatorState := 'WAITING_FOR_DOOR_CLOSE';
        ELSE
            MotorRun := TRUE;
            IF AtFloor[TargetFloor] THEN
                // Stop motor and reset buttons
                MotorRun := FALSE;
                CabinRequest[TargetFloor] := FALSE;
                IF TargetFloor <= 4 THEN UpCall[TargetFloor] := FALSE; END_IF;
                IF TargetFloor >= 2 THEN DownCall[TargetFloor] := FALSE; END_IF;

                // Open door and start timers
                DoorOpen := TRUE;
                DoorClosed := FALSE;
                Timer1(IN := TRUE, PT := T#7S);
                DoorTimerStarted := TRUE;
                State := 4;
            END_IF;
        END_IF;

    3: // REVERSE DIRECTION
        ElevatorState := 'REVERSING';
        GoingUp := NOT GoingUp;
        GoingDown := NOT GoingDown;
        State := 1;

    4: // DOOR OPEN LOGIC
        ElevatorState := 'DOOR_OPEN';
        IF DoorTimerStarted THEN
            Timer1(IN := TRUE, PT := T#7S);
            DoorTimerStarted := FALSE;
        END_IF;

        Timer1(CycleTime_ms := CycleTime_ms);
        IF Timer1.Q THEN
            // After 7s, check for cabin button press
            IF NOT ANY_IN_ARRAY(ADR(CabinRequest[1]), SIZEOF(CabinRequest)) THEN
                Timer2(IN := TRUE, PT := T#10S);
                Timer2(CycleTime_ms := CycleTime_ms);
                IF Timer2.Q THEN
                    DoorOpen := FALSE;
                    DoorClosed := TRUE;
                    Timer1(IN := FALSE);
                    Timer2(IN := FALSE);
                    State := 1; // Return to decision logic
                END_IF;
            ELSE
                DoorOpen := FALSE;
                DoorClosed := TRUE;
                Timer1(IN := FALSE);
                Timer2(IN := FALSE);
                State := 1;
            END_IF;
        END_IF;
END_CASE;

PROGRAM PLC_PRG
VAR
    ElevCtrl : FB_ElevatorController;

    // Simulated inputs
    AtFloor : ARRAY [1..5] OF BOOL := [TRUE, FALSE, FALSE, FALSE, FALSE];
    TopLimit, BottomLimit : BOOL := FALSE;

    CabinReq : ARRAY [1..5] OF BOOL := [FALSE, TRUE, FALSE, FALSE, FALSE];
    UpCalls : ARRAY [1..4] OF BOOL := [FALSE, TRUE, FALSE, FALSE];
    DownCalls : ARRAY [2..5] OF BOOL := [FALSE, FALSE, TRUE, FALSE];

    CycleTime : DINT := 10; // Assume 10ms scan time

    // Outputs
    GoUp, GoDown, RunMotor : BOOL;
    OpenDoor, CloseDoor : BOOL;
    CurrentFloorDisplay : INT;
END_VAR

// Call the controller function block
ElevCtrl(
    AtFloor := AtFloor,
    TopLimit := TopLimit,
    BottomLimit := BottomLimit,
    CabinRequest := CabinReq,
    UpCall := UpCalls,
    DownCall := DownCalls,
    CycleTime_ms := CycleTime
);

// Map outputs
GoUp := ElevCtrl.GoingUp;
GoDown := ElevCtrl.GoingDown;
RunMotor := ElevCtrl.MotorRun;
OpenDoor := ElevCtrl.DoorOpen;
CloseDoor := ElevCtrl.DoorClosed;
CurrentFloorDisplay := ElevCtrl.CurrentFloor;
