FUNCTION_BLOCK FB_EtherCATStateController
VAR_INPUT
    Execute : BOOL := FALSE;           // Rising edge triggers state transition
    Abort : BOOL := FALSE;             // Immediate transition to INIT state
    CurrentState : UINT;               // Input from ESM function block
END_VAR

VAR_OUTPUT
    NextState : UINT;                  // Output to ESM function block
    TransitionActive : BOOL := FALSE;  // Currently executing transition
    StateReached : BOOL := FALSE;      // Target state confirmed
    Error : BOOL := FALSE;             // Transition error detected
    ErrorCode : UINT := 16#0000;       // Detailed error information
END_VAR

VAR
    // State machine enumerations
    ECAT_STATE : (INIT, PREOP, SAFEOP, OP) := INIT;
    
    // Internal control variables
    ExecutePrev : BOOL := FALSE;
    TransitionTimer : TON;
    TargetState : UINT;
    CurrentStateInternal : UINT;
    
    // Constants
    TRANSITION_DELAY : TIME := T#5S;   // Required stabilization delay
    MAX_RETRIES : UINT := 3;           // Maximum transition attempts
    RetryCount : UINT := 0;
END_VAR

// State definitions (EtherCAT standard values)
CONSTANT
    ECAT_STATE_INIT : UINT := 16#01;
    ECAT_STATE_PREOP : UINT := 16#02;
    ECAT_STATE_SAFEOP : UINT := 16#04;
    ECAT_STATE_OP : UINT := 16#08;
END_CONSTANT

// Valid state transitions (EtherCAT specification)
METHOD IsValidTransition : BOOL
VAR_INPUT
    fromState : UINT;
    toState : UINT;
END_VAR
BEGIN
    CASE fromState OF
        ECAT_STATE_INIT:
            RETURN (toState = ECAT_STATE_PREOP);
        
        ECAT_STATE_PREOP:
            RETURN (toState = ECAT_STATE_SAFEOP) OR 
                   (toState = ECAT_STATE_INIT);
        
        ECAT_STATE_SAFEOP:
            RETURN (toState = ECAT_STATE_OP) OR 
                   (toState = ECAT_STATE_PREOP) OR
                   (toState = ECAT_STATE_INIT);
        
        ECAT_STATE_OP:
            RETURN (toState = ECAT_STATE_SAFEOP) OR 
                   (toState = ECAT_STATE_PREOP) OR
                   (toState = ECAT_STATE_INIT);
        
        ELSE
            RETURN FALSE;
    END_CASE;
END_METHOD

// Main state machine control logic
BEGIN
    // Reset outputs each cycle
    StateReached := FALSE;
    
    // Handle abort command (immediate transition to INIT)
    IF Abort THEN
        NextState := ECAT_STATE_INIT;
        TransitionActive := TRUE;
        Error := FALSE;
        RetryCount := 0;
        TransitionTimer(IN := FALSE);
        RETURN;
    END_IF;
    
    // Detect execute rising edge
    IF Execute AND NOT ExecutePrev AND NOT TransitionActive THEN
        // Determine next state in sequence
        CASE CurrentState OF
            ECAT_STATE_INIT:
                TargetState := ECAT_STATE_PREOP;
            
            ECAT_STATE_PREOP:
                TargetState := ECAT_STATE_SAFEOP;
            
            ECAT_STATE_SAFEOP:
                TargetState := ECAT_STATE_OP;
            
            ECAT_STATE_OP:
                // Stay in OP state
                TargetState := ECAT_STATE_OP;
                StateReached := TRUE;
                RETURN;
            
            ELSE
                // Invalid current state
                Error := TRUE;
                ErrorCode := 16#8001;
                RETURN;
        END_CASE;
        
        // Validate the transition
        IF NOT IsValidTransition(CurrentState, TargetState) THEN
            Error := TRUE;
            ErrorCode := 16#8002;
            RETURN;
        END_IF;
        
        // Start transition
        NextState := TargetState;
        TransitionActive := TRUE;
        TransitionTimer(IN := TRUE, PT := TRANSITION_DELAY);
    END_IF;
    
    ExecutePrev := Execute;
    
    // Monitor active transition
    IF TransitionActive THEN
        // Check if target state has been reached
        IF CurrentState = TargetState THEN
            TransitionActive := FALSE;
            TransitionTimer(IN := FALSE);
            StateReached := TRUE;
            Error := FALSE;
            RetryCount := 0;
            
            // Update internal state representation
            CASE TargetState OF
                ECAT_STATE_INIT: ECAT_STATE := INIT;
                ECAT_STATE_PREOP: ECAT_STATE := PREOP;
                ECAT_STATE_SAFEOP: ECAT_STATE := SAFEOP;
                ECAT_STATE_OP: ECAT_STATE := OP;
            END_CASE;
        ELSIF TransitionTimer.Q THEN
            // Transition timeout - retry or error
            IF RetryCount < MAX_RETRIES THEN
                RetryCount := RetryCount + 1;
                TransitionTimer(IN := TRUE, PT := TRANSITION_DELAY);
            ELSE
                // Max retries exceeded
                TransitionActive := FALSE;
                Error := TRUE;
                ErrorCode := 16#8003;
                RetryCount := 0;
            END_IF;
        END_IF;
    END_IF;
END_FUNCTION_BLOCK
