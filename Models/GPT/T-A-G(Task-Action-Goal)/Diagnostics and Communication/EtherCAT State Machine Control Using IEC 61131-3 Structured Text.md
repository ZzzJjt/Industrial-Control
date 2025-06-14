PROGRAM EtherCAT_ESM_Control
VAR
    CurrentState     : STRING[10] := 'INIT';
    NextState        : STRING[10];
    TransitionTimer  : TON;
    TimerStart       : BOOL := FALSE;
    TimerElapsed     : BOOL;
    ErrorFlag        : BOOL := FALSE;
    ErrorMessage     : STRING[80];
END_VAR

(* Timer configuration *)
TransitionTimer(IN := TimerStart, PT := T#5s);
TimerElapsed := TransitionTimer.Q;

(* State transition logic *)
CASE CurrentState OF

    'INIT':
        NextState := 'PREOP';
        TimerStart := TRUE;
        IF TimerElapsed THEN
            CurrentState := NextState;
            TimerStart := FALSE;
        END_IF

    'PREOP':
        IF CheckState('PREOP') THEN
            NextState := 'SAFEOP';
            TimerStart := TRUE;
            IF TimerElapsed THEN
                CurrentState := NextState;
                TimerStart := FALSE;
            END_IF
        ELSE
            ErrorFlag := TRUE;
            ErrorMessage := 'Transition to SAFEOP blocked - invalid PREOP condition';
        END_IF

    'SAFEOP':
        IF CheckState('SAFEOP') THEN
            NextState := 'OP';
            TimerStart := TRUE;
            IF TimerElapsed THEN
                CurrentState := NextState;
                TimerStart := FALSE;
            END_IF
        ELSE
            ErrorFlag := TRUE;
            ErrorMessage := 'Transition to OP blocked - invalid SAFEOP condition';
        END_IF

    'OP':
        IF NOT CheckState('OP') THEN
            ErrorFlag := TRUE;
            ErrorMessage := 'OP state lost unexpectedly';
        END_IF

    ELSE
        ErrorFlag := TRUE;
        ErrorMessage := 'Unknown EtherCAT state';

END_CASE

(* Dummy state validation function *)
FUNCTION CheckState : BOOL
VAR_INPUT
    StateName : STRING;
END_VAR
(* Simulated success for all valid transitions *)
CheckState := (StateName = 'PREOP') OR (StateName = 'SAFEOP') OR (StateName = 'OP');
