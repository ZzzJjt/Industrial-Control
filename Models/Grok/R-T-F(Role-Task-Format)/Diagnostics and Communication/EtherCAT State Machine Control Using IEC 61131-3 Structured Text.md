FUNCTION_BLOCK ETHERCAT_STATEMACHINE
(*
    Function Block: ETHERCAT_STATEMACHINE
    Purpose: Manages EtherCAT State Machine (ESM) transitions for a slave device
             (INIT -> PREOP -> SAFEOP -> OP) with 5-second delays, validation,
             and error handling.
    Standard: IEC 61131-3 Structured Text
*)

(* Symbolic Constants for EtherCAT States *)
VAR CONSTANT
    STATE_INIT   : UINT := 1;  (* Initialization state *)
    STATE_PREOP  : UINT := 2;  (* Pre-Operational state *)
    STATE_SAFEOP : UINT := 4;  (* Safe-Operational state *)
    STATE_OP     : UINT := 8;  (* Operational state *)
END_VAR

(* Input Variables *)
VAR_INPUT
    Start          : BOOL;     (* TRUE to start state machine *)
    Device_Ready   : BOOL;     (* TRUE if device hardware is ready *)
    Comm_Valid     : BOOL;     (* TRUE if communication is valid *)
    System_Tick_ms : UDINT;    (* System tick in milliseconds *)
END_VAR

(* Output Variables *)
VAR_OUTPUT
    Current_State  : UINT;     (* Current ESM state *)
    Error          : BOOL;     (* TRUE if error detected *)
    Error_Code     : UINT;     (* Error code: 0 = No error, 1 = Invalid transition,
                                  2 = Device not ready, 3 = Comm failure *)
    Audit_Log_Entry: STRING[80]; (* Audit log message *)
END_VAR

(* Internal Variables *)
VAR
    (* State machine variables *)
    Target_State    : UINT := STATE_INIT;  (* Desired next state *)
    State_Transition: BOOL := FALSE;       (* TRUE during transition *)
    
    (* Timer for 5-second delay *)
    Delay_Timer     : TON;                 (* 5-second delay timer *)
    Transition_Start_Tick : UDINT;         (* Tick at transition start *)
    
    (* Audit log variables *)
    Last_Log        : STRING[80];          (* Last logged message *)
    
    (* State validation flag *)
    Transition_Valid: BOOL;                (* TRUE if transition is valid *)
END_VAR

(* Internal Methods *)
METHOD PRIVATE ValidateTransition : BOOL
    VAR_INPUT
        From_State : UINT;
        To_State   : UINT;
    END_VAR
    (* Validate allowed transitions per EtherCAT ESM *)
    CASE From_State OF
        STATE_INIT:
            RETURN To_State = STATE_PREOP;
        STATE_PREOP:
            RETURN To_State = STATE_INIT OR To_State = STATE_SAFEOP;
        STATE_SAFEOP:
            RETURN To_State = STATE_PREOP OR To_State = STATE_OP;
        STATE_OP:
            RETURN To_State = STATE_SAFEOP;
        ELSE
            RETURN FALSE;
    END_CASE;
END_METHOD

METHOD PRIVATE LogAuditEntry
    VAR_INPUT
        Message : STRING[80];
    END_VAR
    (* Log audit entry with timestamp and message *)
    Audit_Log_Entry := Message;
    Last_Log := Message;
    (* In practice, append to external log: e.g., CALL LogToHMI(Audit_Log_Entry); *)
END_METHOD

(* Main Execution Logic *)
(* Reset outputs *)
Error := FALSE;
Error_Code := 0;
Audit_Log_Entry := '';

(* Initialize state machine *)
IF Current_State = 0 THEN
    Current_State := STATE_INIT;
    LogAuditEntry('Initialized to INIT state');
END_IF;

(* Handle state machine execution *)
IF Start THEN
    (* Check device and communication status *)
    IF NOT Device_Ready THEN
        Error := TRUE;
        Error_Code := 2;  (* Device not ready *)
        LogAuditEntry('Error: Device not ready');
        RETURN;
    END_IF;
    
    IF NOT Comm_Valid THEN
        Error := TRUE;
        Error_Code := 3;  (* Communication failure *)
        LogAuditEntry('Error: Communication failure');
        RETURN;
    END_IF;
    
    (* Determine target state for sequential progression *)
    IF NOT State_Transition THEN
        CASE Current_State OF
            STATE_INIT:
                Target_State := STATE_PREOP;
            STATE_PREOP:
                Target_State := STATE_SAFEOP;
            STATE_SAFEOP:
                Target_State := STATE_OP;
            STATE_OP:
                Target_State := STATE_OP;  (* Stay in OP *)
            ELSE
                Error := TRUE;
                Error_Code := 1;  (* Invalid transition *)
                LogAuditEntry('Error: Invalid current state');
                RETURN;
        END_CASE;
        
        (* Validate transition *)
        Transition_Valid := ValidateTransition(Current_State, Target_State);
        IF NOT Transition_Valid THEN
            Error := TRUE;
            Error_Code := 1;  (* Invalid transition *)
            LogAuditEntry('Error: Invalid state transition requested');
            RETURN;
        END_IF;
        
        (* Start transition *)
        State_Transition := TRUE;
        Transition_Start_Tick := System_Tick_ms;
        Delay_Timer(IN := TRUE, PT := T#5000ms);
        LogAuditEntry(CONCAT('Transition started: ', 
                             CASE Current_State OF
                                 STATE_INIT: 'INIT to PREOP';
                                 STATE_PREOP: 'PREOP to SAFEOP';
                                 STATE_SAFEOP: 'SAFEOP to OP';
                                 ELSE: 'Unknown';
                             END_CASE));
    END_IF;
    
    (* Handle transition delay *)
    Delay_Timer();
    IF State_Transition AND Delay_Timer.Q THEN
        (* 5-second delay complete, execute transition *)
        Current_State := Target_State;
        State_Transition := FALSE;
        Delay_Timer(IN := FALSE);
        
        (* Log successful transition *)
        CASE Current_State OF
            STATE_PREOP:
                LogAuditEntry('Transition completed: PREOP state');
            STATE_SAFEOP:
                LogAuditEntry('Transition completed: SAFEOP state');
            STATE_OP:
                LogAuditEntry('Transition completed: OP state');
        END_CASE;
        
        (* Simulate EtherCAT stack interaction *)
        (* In practice, call API: e.g., EtherCAT_SetState(Target_State); *)
    END_IF;
ELSE
    (* Stop state machine, reset to INIT *)
    IF Current_State <> STATE_INIT THEN
        Current_State := STATE_INIT;
        State_Transition := FALSE;
        Delay_Timer(IN := FALSE);
        LogAuditEntry('State machine stopped, reset to INIT');
    END_IF;
END_IF;

(* Error handling for timer issues *)
IF State_Transition AND (System_Tick_ms - Transition_Start_Tick > 6000) AND NOT Delay_Timer.Q THEN
    Error := TRUE;
    Error_Code := 1;  (* Invalid transition *)
    LogAuditEntry('Error: Transition timeout');
    State_Transition := FALSE;
    Delay_Timer(IN := FALSE);
END_IF;

END_FUNCTION_BLOCK
