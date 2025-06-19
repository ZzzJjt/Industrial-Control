PROGRAM ETHERCAT_STATE_MACHINE
VAR
    // Symbolic constants for EtherCAT states
    STATE_INIT : INT := 1;
    STATE_PREOP : INT := 2;
    STATE_SAFEOP : INT := 3;
    STATE_OP : INT := 4;

    // Current state of the EtherCAT state machine
    currentState : INT := STATE_INIT;

    // Timer block for 5-second delay
    transitionTimer : TON; // On-Delay Timer
    transitionDelay : TIME := T#5s; // 5 seconds delay

    // Flags for state transitions
    startTransition : BOOL := FALSE;
    transitionComplete : BOOL := FALSE;

    // Error flags and log entries
    errorOccurred : BOOL := FALSE;
    errorMessage : STRING;

    // Output alarms
    alarmInitError : BOOL := FALSE;
    alarmPreopError : BOOL := FALSE;
    alarmSafeopError : BOOL := FALSE;
    alarmOpError : BOOL := FALSE;
END_VAR

// Method to handle state transition
METHOD TransitionToState : BOOL
VAR_INPUT
    newState : INT;
END_VAR
VAR
    validTransition : BOOL := FALSE;
END_VAR

    // Validate the transition based on current state
    CASE currentState OF
        STATE_INIT:
            IF newState = STATE_PREOP THEN
                validTransition := TRUE;
            END_IF;
        STATE_PREOP:
            IF newState = STATE_SAFEOP THEN
                validTransition := TRUE;
            END_IF;
        STATE_SAFEOP:
            IF newState = STATE_OP THEN
                validTransition := TRUE;
            END_IF;
        STATE_OP:
            // No further transitions from OP state in this example
            validTransition := FALSE;
        ELSE
            validTransition := FALSE;
    END_CASE;

    // If transition is valid, start the timer
    IF validTransition THEN
        currentState := newState;
        startTransition := TRUE;
        RETURN TRUE;
    ELSE
        errorMessage := CONCAT("Invalid transition from state ", TO_STRING(currentState), " to state ", TO_STRING(newState));
        LogError(errorMessage);
        RETURN FALSE;
    END_IF;
END_METHOD

// Method to log errors
METHOD LogError : BOOL
VAR_INPUT
    message : STRING;
END_VAR
    errorMessage := message;
    errorOccurred := TRUE;
    RETURN TRUE;
END_METHOD

// Main execution logic
METHOD Execute : BOOL
BEGIN
    // Start the transition timer if a transition is requested
    IF startTransition THEN
        transitionTimer(IN := TRUE, PT := transitionDelay);
        startTransition := FALSE;
    END_IF;

    // Check if the transition timer has completed
    IF transitionTimer.Q THEN
        transitionTimer(IN := FALSE); // Reset the timer
        transitionComplete := TRUE;
    END_IF;

    // Perform actions based on the current state and transition completion
    CASE currentState OF
        STATE_INIT:
            IF transitionComplete THEN
                IF NOT TransitionToState(STATE_PREOP) THEN
                    alarmInitError := TRUE;
                END_IF;
                transitionComplete := FALSE;
            END_IF;
        STATE_PREOP:
            IF transitionComplete THEN
                IF NOT TransitionToState(STATE_SAFEOP) THEN
                    alarmPreopError := TRUE;
                END_IF;
                transitionComplete := FALSE;
            END_IF;
        STATE_SAFEOP:
            IF transitionComplete THEN
                IF NOT TransitionToState(STATE_OP) THEN
                    alarmSafeopError := TRUE;
                END_IF;
                transitionComplete := FALSE;
            END_IF;
        STATE_OP:
            // In OP state, no further transitions
            ; // Do nothing
        ELSE
            errorMessage := CONCAT("Unknown state: ", TO_STRING(currentState));
            LogError(errorMessage);
            errorOccurred := TRUE;
    END_CASE;

    RETURN NOT errorOccurred;
END_METHOD

END_PROGRAM
