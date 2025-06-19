(* State enumeration *)
STATE_INIT : UINT := 1;    (* Initialization state *)
STATE_PREOP : UINT := 2;   (* Pre-Operational state *)
STATE_SAFEOP : UINT := 4;  (* Safe-Operational state *)
STATE_OP : UINT := 8;      (* Operational state *)

(* Program control variables *)
CurrentState : UINT;       (* Current ESM state *)
TargetState : UINT;        (* Desired next state *)
StateTransition : BOOL;    (* Trigger for state change *)
TransitionError : BOOL;    (* TRUE if transition fails *)
ErrorCode : UINT;          (* Error code: 0=None, 1=Invalid Transition, 2=Timeout, 3=Communication Error *)

(* Timer for 5-second stabilization delay *)
StabilizeTimer : TON;      (* Timer for delay *)
STABILIZE_DELAY : TIME := T#5s; (* 5-second delay *)

(* Control flags *)
StartTransition : BOOL;    (* Initiate state transition *)
TransitionComplete : BOOL; (* Transition successfully completed *)
AllStatesReached : BOOL;   (* TRUE when OP state is reached *)


(* Set target state based on current state *)
IF StartTransition AND NOT StabilizeTimer.Q THEN
    CASE CurrentState OF
        0, STATE_INIT:
            TargetState := STATE_PREOP;
            StateTransition := TRUE;
        STATE_PREOP:
            TargetState := STATE_SAFEOP;
            StateTransition := TRUE;
        STATE_SAFEOP:
            TargetState := STATE_OP;
            StateTransition := TRUE;
        STATE_OP:
            AllStatesReached := TRUE;
            StateTransition := FALSE;
        ELSE
            (* Invalid or unexpected state *)
            TransitionError := TRUE;
            ErrorCode := 1; (* Invalid Transition *)
            StateTransition := FALSE;
    END_CASE
END_IF

(* Validate transition *)
IF StateTransition THEN
    (* Check if transition is allowed per EtherCAT spec *)
    IF (CurrentState = STATE_INIT AND TargetState = STATE_PREOP) OR
       (CurrentState = STATE_PREOP AND TargetState = STATE_SAFEOP) OR
       (CurrentState = STATE_SAFEOP AND TargetState = STATE_OP) THEN
        (* Trigger ESM state change *)
        ESM(TargetState := TargetState, Execute := TRUE);
        
        (* Check ESM status *)
        IF ESM.Error THEN
            TransitionError := TRUE;
            ErrorCode := 3; (* Communication Error *)
            StateTransition := FALSE;
            ESM(Execute := FALSE);
        ELSIF ESM.Done THEN
            (* Transition successful, start stabilization timer *)
            StabilizeTimer(IN := TRUE, PT := STABILIZE_DELAY);
            TransitionComplete := TRUE;
            ESM(Execute := FALSE);
            StateTransition := FALSE;
        ELSIF ESM.Timeout THEN
            (* Transition timed out *)
            TransitionError := TRUE;
            ErrorCode := 2; (* Timeout *)
            StateTransition := FALSE;
            ESM(Execute := FALSE);
        END_IF
    ELSE
        (* Invalid transition *)
        TransitionError := TRUE;
        ErrorCode := 1; (* Invalid Transition *)
        StateTransition := FALSE;
    END_IF
END_IF

(* Handle stabilization delay *)
IF TransitionComplete AND StabilizeTimer.Q THEN
    (* Reset for next transition *)
    TransitionComplete := FALSE;
    StabilizeTimer(IN := FALSE);
    StartTransition := FALSE;
    (* Trigger next transition *)
    IF CurrentState <> STATE_OP THEN
        StartTransition := TRUE;
    END_IF
END_IF
