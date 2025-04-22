```
PROGRAM EtherCATStateMachineControl
VAR
    // Timer variables
    TimerInitPreop : TON; // Timer from INIT to PREOP
    TimerPreopSafeop : TON; // Timer from PREOP to SAFEOP
    TimerSafeopOp : TON; // Timer from SAFEOP to OP
    
    // State variables
    CurrentState : ('INIT', 'PREOP', 'SAFEOP', 'OP');
    
    // Flags for state transitions
    TransitionInitToPreop : BOOL := FALSE;
    TransitionPreopToSafeop : BOOL := FALSE;
    TransitionSafeopToOp : BOOL := FALSE;
    
    // Error handling
    ErrorOccurred : BOOL := FALSE;
BEGIN
    // Initial state
    CurrentState := 'INIT';
    
    // Handle state transitions
    CASE CurrentState OF
        'INIT':
            TimerInitPreop(IN := TRUE, PT := T#5s);
            IF TimerInitPreOp.Q THEN
                TransitionInitToPreop := TRUE;
            END_IF
            
        'PREOP':
            TimerPreopSafeop(IN := TRUE, PT := T#5s);
            IF TimerPreopSafeop.Q THEN
                TransitionPreopToSafeop := TRUE;
            END_IF
            
        'SAFEOP':
            TimerSafeopOp(IN := TRUE, PT := T#5s);
            IF TimerSafeopOp.Q THEN
                TransitionSafeopToOp := TRUE;
            END_IF
            
        'OP':
            // In OP state, no further transition planned
            // Implement operational logic here
    END_CASE
    
    // Transition logic
    IF TransitionInitToPreop THEN
        CurrentState := 'PREOP';
        TransitionInitToPreop := FALSE;
    END_IF
    
    IF TransitionPreopToSafeop THEN
        CurrentState := 'SAFEOP';
        TransitionPreopToSafeop := FALSE;
    END_IF
    
    IF TransitionSafeopToOp THEN
        CurrentState := 'OP';
        TransitionSafeopToOp := FALSE;
    END_IF
    
    // Error handling
    IF ErrorOccurred THEN
        // Implement error recovery or logging here
    END_IF
    
END_PROGRAM
```
