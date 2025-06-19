// ================================================
// FUNCTION_BLOCK EtherCAT_StateManager
// Controls EtherCAT Slave transitions: INIT → PREOP → SAFEOP → OP
// ================================================

FUNCTION_BLOCK EtherCAT_StateManager
VAR_INPUT
    StartTransition : BOOL;       // Trigger to start state machine
    CurrentState    : STRING(10); // External readback of actual state (e.g., from slave feedback)
END_VAR

VAR_OUTPUT
    TargetState     : STRING(10); // Target state to command
    ErrorDetected   : BOOL;       // TRUE if an invalid transition or fault occurs
    AlarmMessage    : STRING(255);// Description of the detected error
END_VAR

VAR
    StepIndex       : INT := 0;   // Step tracker
    TransitionTimer : TON;       // 5s delay between transitions
    TransitionEnable: BOOL := FALSE;

    CONST
        STATE_INIT    : STRING := 'INIT';
        STATE_PREOP   : STRING := 'PREOP';
        STATE_SAFEOP  : STRING := 'SAFEOP';
        STATE_OP      : STRING := 'OP';
END_VAR

// ====================================================
// STATE MACHINE LOGIC
// ====================================================
TransitionTimer(IN := TransitionEnable, PT := T#5s);

// Start transition process
IF StartTransition AND StepIndex = 0 THEN
    TargetState := STATE_INIT;
    TransitionEnable := TRUE;
    StepIndex := 1;
END_IF

CASE StepIndex OF

    1: // Wait 5s after INIT
        IF TransitionTimer.Q THEN
            IF CurrentState = STATE_INIT THEN
                TargetState := STATE_PREOP;
                TransitionEnable := TRUE;
                TransitionTimer(IN := FALSE);
                StepIndex := 2;
            ELSE
                ErrorDetected := TRUE;
                AlarmMessage := 'Transition to INIT failed.';
            END_IF
        END_IF

    2: // Wait 5s after PREOP
        IF TransitionTimer.Q THEN
            IF CurrentState = STATE_PREOP THEN
                TargetState := STATE_SAFEOP;
                TransitionEnable := TRUE;
                TransitionTimer(IN := FALSE);
                StepIndex := 3;
            ELSE
                ErrorDetected := TRUE;
                AlarmMessage := 'Transition to PREOP failed.';
            END_IF
        END_IF

    3: // Wait 5s after SAFEOP
        IF TransitionTimer.Q THEN
            IF CurrentState = STATE_SAFEOP THEN
                TargetState := STATE_OP;
                TransitionEnable := TRUE;
                TransitionTimer(IN := FALSE);
                StepIndex := 4;
            ELSE
                ErrorDetected := TRUE;
                AlarmMessage := 'Transition to SAFEOP failed.';
            END_IF
        END_IF

    4: // Final state check
        IF TransitionTimer.Q THEN
            IF CurrentState = STATE_OP THEN
                TransitionEnable := FALSE;
                StepIndex := 5; // Done
            ELSE
                ErrorDetected := TRUE;
                AlarmMessage := 'Transition to OP failed.';
            END_IF
        END_IF

    ELSE
        // Idle or completed
END_CASE

// Timer Reset Management
IF NOT TransitionEnable THEN
    TransitionTimer(IN := FALSE);
ELSE
    TransitionTimer(IN := TRUE);
END_IF
