(* IEC 61131-3 Structured Text function block for EtherCAT State Machine control *)
(* Manages slave transitions through INIT, PREOP, SAFEOP, OP states *)
(* Ensures 5-second delays, valid transitions, and robust error handling *)

FUNCTION_BLOCK ETHERCAT_SLAVE_CONTROL
VAR_INPUT
    ENABLE : BOOL; (* TRUE to enable function block *)
    START : BOOL; (* TRUE to initiate state transitions *)
    CURRENT_STATE : DWORD; (* Current ESM state from slave *)
    TRANSITION_ACK : BOOL; (* TRUE if slave acknowledges transition *)
END_VAR

VAR_OUTPUT
    DONE : BOOL; (* TRUE when OP state is reached *)
    BUSY : BOOL; (* TRUE during transition *)
    ERROR : BOOL; (* TRUE if error occurs *)
    ERROR_ID : DWORD; (* Error code: 0=None, 1=Invalid State, 2=Timeout, 3=Transition Failed *)
    REQUESTED_STATE : DWORD; (* Target ESM state *)
END_VAR

VAR
    (* Symbolic EtherCAT states *)
    ESM_INIT : DWORD := 16#01;
    ESM_PREOP : DWORD := 16#02;
    ESM_SAFEOP : DWORD := 16#04;
    ESM_OP : DWORD := 16#08;
    
    (* Internal state machine *)
    CurrentStep : INT := 0; (* 0=Idle, 1=INIT, 2=PREOP, 3=SAFEOP, 4=OP *)
    TransitionTimer : TON; (* 5-second delay timer *)
    LastEnable : BOOL; (* Tracks ENABLE state *)
    LastStart : BOOL; (* Tracks START state *)
    OperationActive : BOOL; (* Tracks ongoing operation *)
END_VAR

(* Reset outputs when disabled *)
IF NOT ENABLE THEN
    DONE := FALSE;
    BUSY := FALSE;
    ERROR := FALSE;
    ERROR_ID := 0;
    REQUESTED_STATE := 0;
    CurrentStep := 0;
    TransitionTimer(IN := FALSE);
    OperationActive := FALSE;
    LastEnable := FALSE;
    LastStart := FALSE;
    RETURN;
END_IF;

(* Initialize on rising edge of ENABLE *)
IF ENABLE AND NOT LastEnable THEN
    (* Reset state machine *)
    CurrentStep := 0;
    DONE := FALSE;
    BUSY := FALSE;
    ERROR := FALSE;
    ERROR_ID := 0;
    REQUESTED_STATE := 0;
    OperationActive := FALSE;
END_IF;

(* Store ENABLE state *)
LastEnable := ENABLE;

(* Main state machine *)
CASE CurrentStep OF
    0: (* Idle *)
        IF START AND NOT LastStart AND NOT OperationActive THEN
            (* Start transition sequence *)
            CurrentStep := 1;
            BUSY := TRUE;
            DONE := FALSE;
            ERROR := FALSE;
            ERROR_ID := 0;
            OperationActive := TRUE;
            TransitionTimer(IN := FALSE);
        END_IF;
    
    1: (* INIT *)
        (* Request INIT state *)
        REQUESTED_STATE := ESM_INIT;
        
        (* Validate current state *)
        IF CURRENT_STATE <> ESM_INIT THEN
            ERROR := TRUE;
            ERROR_ID := 1; (* Invalid State *)
            BUSY := FALSE;
            OperationActive := FALSE;
            CurrentStep := 0;
            RETURN;
        END_IF;
        
        (* Start 5-second timer *)
        TransitionTimer(IN := TRUE, PT := T#5s);
        IF TransitionTimer.Q THEN
            (* Check acknowledgment *)
            IF TRANSITION_ACK THEN
                CurrentStep := 2; (* Move to PREOP *)
                TransitionTimer(IN := FALSE);
            ELSE
                ERROR := TRUE;
                ERROR_ID := 2; (* Timeout *)
                BUSY := FALSE;
                OperationActive := FALSE;
                CurrentStep := 0;
            END_IF;
        END_IF;
    
    2: (* PREOP *)
        (* Request PREOP state *)
        REQUESTED_STATE := ESM_PREOP;
        
        (* Validate transition *)
        IF CURRENT_STATE <> ESM_INIT AND CURRENT_STATE <> ESM_PREOP THEN
            ERROR := TRUE;
            ERROR_ID := 1; (* Invalid State *)
            BUSY := FALSE;
            OperationActive := FALSE;
            CurrentStep := 0;
            RETURN;
        END_IF;
        
        (* Start timer *)
        TransitionTimer(IN := TRUE, PT := T#5s);
        IF TransitionTimer.Q THEN
            IF TRANSITION_ACK AND CURRENT_STATE = ESM_PREOP THEN
                CurrentStep := 3; (* Move to SAFEOP *)
                TransitionTimer(IN := FALSE);
            ELSE
                ERROR := TRUE;
                ERROR_ID := 3; (* Transition Failed *)
                BUSY := FALSE;
                OperationActive := FALSE;
                CurrentStep := 0;
            END_IF;
        END_IF;
    
    3: (* SAFEOP *)
        (* Request SAFEOP state *)
        REQUESTED_STATE := ESM_SAFEOP;
        
        (* Validate transition *)
        IF CURRENT_STATE <> ESM_PREOP AND CURRENT_STATE <> ESM_SAFEOP THEN
            ERROR := TRUE;
            ERROR_ID := 1; (* Invalid State *)
            BUSY := FALSE;
            OperationActive := FALSE;
            CurrentStep := 0;
            RETURN;
        END_IF;
        
        (* Start timer *)
        TransitionTimer(IN := TRUE, PT := T#5s);
        IF TransitionTimer.Q THEN
            IF TRANSITION_ACK AND CURRENT_STATE = ESM_SAFEOP THEN
                CurrentStep := 4; (* Move to OP *)
                TransitionTimer(IN := FALSE);
            ELSE
                ERROR := TRUE;
                ERROR_ID := 3; (* Transition Failed *)
                BUSY := FALSE;
                OperationActive := FALSE;
                CurrentStep := 0;
            END_IF;
        END_IF;
    
    4: (* OP *)
        (* Request OP state *)
        REQUESTED_STATE := ESM_OP;
        
        (* Validate transition *)
        IF CURRENT_STATE <> ESM_SAFEOP AND CURRENT_STATE <> ESM_OP THEN
            ERROR := TRUE;
            ERROR_ID := 1; (* Invalid State *)
            BUSY := FALSE;
            OperationActive := FALSE;
            CurrentStep := 0;
            RETURN;
        END_IF;
        
        (* Start timer *)
        TransitionTimer(IN := TRUE, PT := T#5s);
        IF TransitionTimer.Q THEN
            IF TRANSITION_ACK AND CURRENT_STATE = ESM_OP THEN
                DONE := TRUE;
                BUSY := FALSE;
                OperationActive := FALSE;
                CurrentStep := 0;
            ELSE
                ERROR := TRUE;
                ERROR_ID := 3; (* Transition Failed *)
                BUSY := FALSE;
                OperationActive := FALSE;
                CurrentStep := 0;
            END_IF;
            TransitionTimer(IN := FALSE);
        END_IF;
END_CASE;

(* Execution Notes *)
(* - Uses symbolic state names (ESM_INIT, ESM_PREOP, ESM_SAFEOP, ESM_OP) for readability *)
(* - Implements 5-second TON timer delays for stable transitions *)
(* - Validates transitions per EtherCAT protocol (ETG.1000.6) *)
(* - Error handling covers invalid states, timeouts, and failed transitions *)
(* - Executes in a single PLC scan cycle, suitable for real-time EtherCAT networks *)
(* - Maintainable code with clear state machine and diagnostic outputs *)
END_FUNCTION_BLOCK
