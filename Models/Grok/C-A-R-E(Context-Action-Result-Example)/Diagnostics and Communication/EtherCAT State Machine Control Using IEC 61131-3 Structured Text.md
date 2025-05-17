(* IEC 61131-3 Structured Text: ETHERCAT_STATE_MACHINE Function Block *)
(* Purpose: Controls EtherCAT State Machine transitions for a slave device *)

FUNCTION_BLOCK ETHERCAT_STATE_MACHINE
VAR_INPUT
    ENABLE : BOOL;                  (* TRUE to start ESM *)
    SLAVE_READY : BOOL;             (* TRUE if slave is ready for transition *)
    RESET : BOOL;                   (* TRUE to reset to INIT state *)
END_VAR
VAR_OUTPUT
    CURRENT_STATE : STRING[10];     (* Current state: INIT, PREOP, SAFEOP, OP *)
    DONE : BOOL;                    (* TRUE when OP state is reached *)
    ERROR : BOOL;                   (* TRUE if error occurs *)
    ERROR_CODE : INT;               (* 0: No error, 1: Invalid transition, 2: Slave not ready, 3: Timeout *)
END_VAR
VAR
    State : INT := 0;               (* State: 0=INIT, 1=PREOP, 2=SAFEOP, 3=OP *)
    PrevState : INT;                (* Previous state for validation *)
    Timer : TON;                    (* 5-second delay timer *)
    TimeoutTimer : TON;             (* 10-second timeout timer *)
    TransitionRequested : BOOL;      (* TRUE when transition is pending *)
    StartTime : TIME;               (* Transition start time *)
END_VAR

(* Main Logic *)
IF RESET THEN
    (* Reset to INIT state *)
    State := 0;
    CURRENT_STATE := 'INIT';
    DONE := FALSE;
    ERROR := FALSE;
    ERROR_CODE := 0;
    TransitionRequested := FALSE;
    Timer.IN := FALSE;
    TimeoutTimer.IN := FALSE;
ELSIF NOT ENABLE THEN
    (* Disable: Hold INIT, reset outputs *)
    State := 0;
    CURRENT_STATE := 'INIT';
    DONE := FALSE;
    ERROR := FALSE;
    ERROR_CODE := 0;
    TransitionRequested := FALSE;
    Timer.IN := FALSE;
    TimeoutTimer.IN := FALSE;
ELSE
    CASE State OF
        0: (* INIT *)
            CURRENT_STATE := 'INIT';
            IF NOT TransitionRequested THEN
                (* Start transition to PREOP *)
                TransitionRequested := TRUE;
                Timer(IN := TRUE, PT := T#5s);
                TimeoutTimer(IN := TRUE, PT := T#10s);
                StartTime := TIME();
            END_IF;
            IF Timer.Q THEN
                IF SLAVE_READY THEN
                    State := 1; (* Move to PREOP *)
                    PrevState := 0;
                    TransitionRequested := FALSE;
                    Timer.IN := FALSE;
                    TimeoutTimer.IN := FALSE;
                ELSE
                    ERROR := TRUE;
                    ERROR_CODE := 2; (* Slave not ready *)
                END_IF;
            ELSIF TimeoutTimer.Q THEN
                ERROR := TRUE;
                ERROR_CODE := 3; (* Timeout *)
                TransitionRequested := FALSE;
                Timer.IN := FALSE;
                TimeoutTimer.IN := FALSE;
            END_IF;
        
        1: (* PREOP *)
            CURRENT_STATE := 'PREOP';
            IF NOT TransitionRequested THEN
                (* Start transition to SAFEOP *)
                TransitionRequested := TRUE;
                Timer(IN := TRUE, PT := T#5s);
                TimeoutTimer(IN := TRUE, PT := T#10s);
                StartTime := TIME();
            END_IF;
            IF Timer.Q THEN
                IF SLAVE_READY THEN
                    State := 2; (* Move to SAFEOP *)
                    PrevState := 1;
                    TransitionRequested := FALSE;
                    Timer.IN := FALSE;
                    TimeoutTimer.IN := FALSE;
                ELSE
                    ERROR := TRUE;
                    ERROR_CODE := 2; (* Slave not ready *)
                END_IF;
            ELSIF TimeoutTimer.Q THEN
                ERROR := TRUE;
                ERROR_CODE := 3; (* Timeout *)
                TransitionRequested := FALSE;
                Timer.IN := FALSE;
                TimeoutTimer.IN := FALSE;
            END_IF;
        
        2: (* SAFEOP *)
            CURRENT_STATE := 'SAFEOP';
            IF NOT TransitionRequested THEN
                (* Start transition to OP *)
                TransitionRequested := TRUE;
                Timer(IN := TRUE, PT := T#5s);
                TimeoutTimer(IN := TRUE, PT := T#10s);
                StartTime := TIME();
            END_IF;
            IF Timer.Q THEN
                IF SLAVE_READY THEN
                    State := 3; (* Move to OP *)
                    PrevState := 2;
                    TransitionRequested := FALSE;
                    Timer.IN := FALSE;
                    TimeoutTimer.IN := FALSE;
                    DONE := TRUE;
                ELSE
                    ERROR := TRUE;
                    ERROR_CODE := 2; (* Slave not ready *)
                END_IF;
            ELSIF TimeoutTimer.Q THEN
                ERROR := TRUE;
                ERROR_CODE := 3; (* Timeout *)
                TransitionRequested := FALSE;
                Timer.IN := FALSE;
                TimeoutTimer.IN := FALSE;
            END_IF;
        
        3: (* OP *)
            CURRENT_STATE := 'OP';
            DONE := TRUE;
            (* Remain in OP unless RESET or disabled *)
    END_CASE;
    
    (* Transition Validation *)
    IF State > PrevState + 1 THEN
        ERROR := TRUE;
        ERROR_CODE := 1; (* Invalid transition *)
        State := PrevState; (* Revert to previous state *)
        TransitionRequested := FALSE;
        Timer.IN := FALSE;
        TimeoutTimer.IN := FALSE;
    END_IF;
END_IF;

(* Notes:
   - Purpose: Controls EtherCAT State Machine for a slave device (INIT → PREOP → SAFEOP → OP).
   - Inputs:
     - ENABLE: TRUE to start ESM.
     - SLAVE_READY: TRUE if slave is ready for transition (e.g., driver feedback).
     - RESET: TRUE to reset to INIT state.
   - Outputs:
     - CURRENT_STATE: Symbolic state ("INIT", "PREOP", "SAFEOP", "OP").
     - DONE: TRUE when OP state is reached.
     - ERROR: TRUE if error occurs.
     - ERROR_CODE: 0 (No error), 1 (Invalid transition), 2 (Slave not ready), 3 (Timeout).
   - Logic:
     - Transitions through INIT (0), PREOP (1), SAFEOP (2), OP (3) with 5-second delays.
     - Verifies SLAVE_READY before each transition.
     - Implements 10-second timeout for each transition.
     - Uses symbolic state names for readability.
   - Error Handling:
     - Invalid transition: State > PrevState + 1 (ERROR_CODE=1).
     - Slave not ready: SLAVE_READY=FALSE during transition (ERROR_CODE=2).
     - Timeout: Transition exceeds 10 seconds (ERROR_CODE=3).
     - Halts on error, awaits RESET or disable.
   - Safety:
     - State machine ensures scan-cycle safety.
     - Bounded timers (T#5s, T#10s) prevent infinite waits.
     - RESET returns to INIT, clearing errors.
   - Usage:
     - Post-power-up: Transitions INIT → PREOP → SAFEOP → OP, 5s per state, if SLAVE_READY.
     - Error: Logs "Slave not ready" or "Timeout", halts for operator action.
     - Reset: RESET=TRUE reinitializes to INIT.
   - Maintenance:
     - Symbolic names (INIT, PREOP, etc.) improve readability.
     - ERROR_CODE supports HMI diagnostics.
   - Platform Notes:
     - Assumes TON timer and STRING[10] support.
     - SLAVE_READY is driver-provided; replace with actual EtherCAT status.
     - 5-second delay is configurable; adjust PT for specific slaves.
*)
END_FUNCTION_BLOCK
