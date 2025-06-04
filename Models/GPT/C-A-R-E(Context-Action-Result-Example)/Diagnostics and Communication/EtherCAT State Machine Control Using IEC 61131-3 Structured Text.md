PROGRAM EtherCAT_StateManager
VAR
    State         : STRING[10] := 'INIT';         // Current ESM state
    NextState     : STRING[10];
    StateTimer    : TON;                          // Timer for delay between transitions
    TimerTrigger  : BOOL := FALSE;
    TransitionOK  : BOOL := FALSE;                // Simulated transition success flag
    ErrorFlag     : BOOL := FALSE;                // Error flag if transition fails
END_VAR

// Simulated function block to apply and verify EtherCAT state transition
FUNCTION_BLOCK FB_ApplyState
VAR_INPUT
    TargetState : STRING[10];
END_VAR
VAR_OUTPUT
    Success : BOOL;
END_VAR
VAR
END_VAR

// Simulate success for illustration (replace with actual communication in real system)
Success := TRUE;

END_FUNCTION_BLOCK

VAR
    StateApplier : FB_ApplyState;
END_VAR

// --- Main Logic ---
IF NOT ErrorFlag THEN
    CASE State OF
        'INIT':
            TimerTrigger := TRUE;
            StateTimer(IN := TimerTrigger, PT := T#5s);
            IF StateTimer.Q THEN
                StateApplier(TargetState := 'PREOP');
                IF StateApplier.Success THEN
                    State := 'PREOP';
                    TimerTrigger := FALSE;
                ELSE
                    ErrorFlag := TRUE;
                END_IF
            END_IF

        'PREOP':
            TimerTrigger := TRUE;
            StateTimer(IN := TimerTrigger, PT := T#5s);
            IF StateTimer.Q THEN
                StateApplier(TargetState := 'SAFEOP');
                IF StateApplier.Success THEN
                    State := 'SAFEOP';
                    TimerTrigger := FALSE;
                ELSE
                    ErrorFlag := TRUE;
                END_IF
            END_IF

        'SAFEOP':
            TimerTrigger := TRUE;
            StateTimer(IN := TimerTrigger, PT := T#5s);
            IF StateTimer.Q THEN
                StateApplier(TargetState := 'OP');
                IF StateApplier.Success THEN
                    State := 'OP';
                    TimerTrigger := FALSE;
                ELSE
                    ErrorFlag := TRUE;
                END_IF
            END_IF

        'OP':
            // Final operational state reached
            TimerTrigger := FALSE;

        ELSE
            ErrorFlag := TRUE;
    END_CASE
ELSE
    // Log or handle the error
    // (could trigger alarm, write to log, notify operator, etc.)
END_IF
