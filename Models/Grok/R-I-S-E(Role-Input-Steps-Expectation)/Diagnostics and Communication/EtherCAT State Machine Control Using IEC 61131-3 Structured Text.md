FUNCTION_BLOCK ETHERCAT_STATE_MACHINE
VAR_INPUT
    EXECUTE: BOOL;             // Cyclic execution trigger
    REQUESTED_STATE: INT;      // Desired state (1=INIT, 2=PREOP, 4=SAFEOP, 8=OP)
    DEVICE_READY: BOOL;        // TRUE if device is ready for transition
    DEVICE_ERROR: INT;         // Device error code (0 = no error)
END_VAR

VAR_OUTPUT
    CURRENT_STATE: INT;        // Current state (1=INIT, 2=PREOP, 4=SAFEOP, 8=OP)
    DONE: BOOL;                // TRUE when transition completes
    ERROR: BOOL;               // TRUE if error occurs
    ERROR_CODE: INT;           // 0: No error, 1: Invalid transition, 2: Device not ready, 3: Device error
    AUDIT_MESSAGE: STRING[80]; // Transition or error details
END_VAR

VAR
    // State enumeration
    TYPE ESM_STATE:
    (
        STATE_INIT := 1,
        STATE_PREOP := 2,
        STATE_SAFEOP := 4,
        STATE_OP := 8
    );
    END_TYPE
    
    TransitionTimer: TON;      // 5-second delay timer
    TimerStarted: BOOL;        // Tracks if timer is running
    TargetState: ESM_STATE;    // Temporary target state during transition
    ExecuteEdge: BOOL;         // Edge detection for EXECUTE
    PreviousState: ESM_STATE;  // Previous state for audit
END_VAR

// Reset outputs when not executing
IF NOT EXECUTE THEN
    DONE := FALSE;
    ERROR := FALSE;
    ERROR_CODE := 0;
    AUDIT_MESSAGE := '';
    TransitionTimer(IN := FALSE);
    TimerStarted := FALSE;
    ExecuteEdge := FALSE;
    RETURN;
END_IF;

// Cyclic execution
IF EXECUTE THEN
    // Initialize on first cycle
    IF NOT ExecuteEdge THEN
        ExecuteEdge := TRUE;
        CURRENT_STATE := STATE_INIT; // Start in INIT
        PreviousState := STATE_INIT;
        AUDIT_MESSAGE := 'Initialized in INIT state';
    END_IF;
    
    // Reset outputs for this cycle
    DONE := FALSE;
    ERROR := FALSE;
    ERROR_CODE := 0;
    
    // Validate REQUESTED_STATE
    IF REQUESTED_STATE <> STATE_INIT AND
       REQUESTED_STATE <> STATE_PREOP AND
       REQUESTED_STATE <> STATE_SAFEOP AND
       REQUESTED_STATE <> STATE_OP THEN
        ERROR := TRUE;
        ERROR_CODE := 1; // Invalid transition
        AUDIT_MESSAGE := 'Invalid REQUESTED_STATE';
        RETURN;
    END_IF;
    
    // Check device readiness and errors
    IF NOT DEVICE_READY THEN
        ERROR := TRUE;
        ERROR_CODE := 2; // Device not ready
        AUDIT_MESSAGE := 'Device not ready for transition';
        RETURN;
    END_IF;
    
    IF DEVICE_ERROR <> 0 THEN
        ERROR := TRUE;
        ERROR_CODE := 3; // Device error
        AUDIT_MESSAGE := 'Device error detected: ' + INT_TO_STRING(DEVICE_ERROR);
        RETURN;
    END_IF;
    
    // Handle state transition if requested state differs
    IF REQUESTED_STATE <> CURRENT_STATE AND NOT TimerStarted THEN
        // Validate transition
        CASE CURRENT_STATE OF
            STATE_INIT:
                IF REQUESTED_STATE <> STATE_PREOP THEN
                    ERROR := TRUE;
                    ERROR_CODE := 1; // Invalid transition
                    AUDIT_MESSAGE := 'Invalid transition from INIT';
                    RETURN;
                END_IF;
            STATE_PREOP:
                IF REQUESTED_STATE <> STATE_SAFEOP AND REQUESTED_STATE <> STATE_INIT THEN
                    ERROR := TRUE;
                    ERROR_CODE := 1; // Invalid transition
                    AUDIT_MESSAGE := 'Invalid transition from PREOP';
                    RETURN;
                END_IF;
            STATE_SAFEOP:
                IF REQUESTED_STATE <> STATE_OP AND REQUESTED_STATE <> STATE_PREOP THEN
                    ERROR := TRUE;
                    ERROR_CODE := 1; // Invalid transition
                    AUDIT_MESSAGE := 'Invalid transition from SAFEOP';
                    RETURN;
                END_IF;
            STATE_OP:
                IF REQUESTED_STATE <> STATE_SAFEOP THEN
                    ERROR := TRUE;
                    ERROR_CODE := 1; // Invalid transition
                    AUDIT_MESSAGE := 'Invalid transition from OP';
                    RETURN;
                END_IF;
            ELSE:
                ERROR := TRUE;
                ERROR_CODE := 1; // Invalid current state
                AUDIT_MESSAGE := 'Unexpected CURRENT_STATE';
                RETURN;
        END_CASE;
        
        // Start 5-second delay
        TargetState := REQUESTED_STATE;
        TransitionTimer(IN := TRUE, PT := T#5s);
        TimerStarted := TRUE;
        AUDIT_MESSAGE := 'Transition started from ' + INT_TO_STRING(CURRENT_STATE) + 
                         ' to ' + INT_TO_STRING(TargetState);
    END_IF;
    
    // Check timer and complete transition
    IF TimerStarted AND TransitionTimer.Q THEN
        PreviousState := CURRENT_STATE;
        CURRENT_STATE := TargetState;
        TimerStarted := FALSE;
        TransitionTimer(IN := FALSE);
        DONE := TRUE;
        AUDIT_MESSAGE := 'Transition completed to ' + INT_TO_STRING(CURRENT_STATE);
    END_IF;
    
    // Update audit message if no transition is in progress
    IF NOT TimerStarted AND NOT DONE THEN
        AUDIT_MESSAGE := 'Stable in state ' + INT_TO_STRING(CURRENT_STATE);
    END_IF;
END_IF;
END_FUNCTION_BLOCK
