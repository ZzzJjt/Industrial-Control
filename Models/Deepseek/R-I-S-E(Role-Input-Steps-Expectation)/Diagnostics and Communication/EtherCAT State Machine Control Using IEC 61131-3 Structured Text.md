TYPE ESM_STATE :
(
    INIT = 0,
    PREOP = 1,
    SAFEOP = 2,
    OP = 3
);
END_TYPE

FUNCTION_BLOCK FB_EtherCAT_StateMachine
VAR_INPUT
    REQUEST_NEXT_STATE: BOOL := FALSE; // Trigger next valid transition
    RESET_MACHINE: BOOL := FALSE;      // Reset machine to INIT
END_VAR

VAR_OUTPUT
    CURRENT_STATE: ESM_STATE := INIT;
    TRANSITION_IN_PROGRESS: BOOL := FALSE;
    ERROR_OCCURRED: BOOL := FALSE;
    ERROR_CODE: BYTE := 0;              // 0 = No error
END_VAR

VAR
    nextState: ESM_STATE := INIT;
    stateTimer: TON;                    // Timer for delay enforcement
    timerStarted: BOOL := FALSE;
    lastValidState: ESM_STATE := INIT;
END_VAR

// Reset logic
IF RESET_MACHINE THEN
    CURRENT_STATE := INIT;
    lastValidState := INIT;
    REQUEST_NEXT_STATE := FALSE;
    ERROR_OCCURRED := FALSE;
    ERROR_CODE := 0;
END_IF;

// Determine next expected state
CASE CURRENT_STATE OF
    INIT:
        nextState := PREOP;
    PREOP:
        nextState := SAFEOP;
    SAFEOP:
        nextState := OP;
    OP:
        nextState := OP; // Final state - no further transitions
    ELSE
        nextState := INIT;
END_CASE;

// Handle request for next state
IF REQUEST_NEXT_STATE AND NOT timerStarted THEN
    IF IsTransitionValid(CURRENT_STATE, nextState) THEN
        stateTimer(IN := TRUE, PT := T#5s); // Start 5-second delay
        TRANSITION_IN_PROGRESS := TRUE;
        timerStarted := TRUE;
        ERROR_OCCURRED := FALSE;
    ELSE
        ERROR_OCCURRED := TRUE;
        ERROR_CODE := 1; // Invalid transition
    END_IF;
END_IF;

// Check timer expiration
IF stateTimer.Q THEN
    CURRENT_STATE := nextState;
    lastValidState := nextState;
    stateTimer(IN := FALSE); // Stop timer
    TRANSITION_IN_PROGRESS := FALSE;
    timerStarted := FALSE;
    REQUEST_NEXT_STATE := FALSE;
END_IF;

METHOD PRIVATE IsTransitionValid: BOOL
VAR_INPUT
    currentState: ESM_STATE;
    targetState: ESM_STATE;
END_VAR

CASE currentState OF
    INIT:
        IsTransitionValid := (targetState = PREOP);
    PREOP:
        IsTransitionValid := (targetState = SAFEOP);
    SAFEOP:
        IsTransitionValid := (targetState = OP);
    OP:
        IsTransitionValid := FALSE; // No forward transitions from OP
    ELSE
        IsTransitionValid := FALSE;
END_CASE;

FUNCTION GetErrorDescription : STRING(80)
VAR_INPUT
    errorCode: BYTE;
END_VAR

CASE errorCode OF
    0: GetErrorDescription := 'No Error';
    1: GetErrorDescription := 'Invalid Transition Requested';
    2: GetErrorDescription := 'Unknown Current State';
    3: GetErrorDescription := 'Timeout During Transition';
    ELSE: GetErrorDescription := 'Unknown Error';
END_CASE;
