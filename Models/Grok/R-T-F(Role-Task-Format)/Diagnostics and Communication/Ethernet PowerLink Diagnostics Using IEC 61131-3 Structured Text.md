FUNCTION_BLOCK EPL_DIAG_RETRIEVER
(*
    Function Block: EPL_DIAG_RETRIEVER
    Purpose: Retrieves diagnostic data (communication status, error codes, node health)
             from Ethernet PowerLink control nodes via the Managing Node (MN).
             Runs cyclically, processes data, and handles errors.
    Standard: IEC 61131-3 Structured Text
    Date: May 20, 2025
*)

(* Input Variables *)
VAR_INPUT
    Node_Address     : USINT;     (* EPL node address (1..240) *)
    Request_Trigger  : BOOL;      (* R_EDGE: Trigger diagnostic data request *)
    Enable           : BOOL;      (* TRUE: Enable cyclic operation *)
    System_Tick_ms   : UDINT;     (* System tick in milliseconds for timing *)
END_VAR

(* Output Variables *)
VAR_OUTPUT
    Comm_Status      : BOOL;      (* TRUE: Node communication active *)
    Error_Code       : UINT;      (* EPL error code: 0 = No error *)
    Node_Health      : USINT;     (* Health status: 0 = OK, 1 = Warning, 2 = Critical *)
    Diag_Valid       : BOOL;      (* TRUE: Diagnostic data is valid *)
    Error            : BOOL;      (* TRUE: Error occurred *)
    Error_Desc       : STRING[50]; (* Error description *)
    Audit_Log_Entry  : STRING[80]; (* Audit log message *)
END_VAR

(* Internal Variables *)
VAR
    (* Diagnostic data structure *)
    TYPE DiagData :
        STRUCT
            Comm_Active : BOOL;     (* Node communication status *)
            Error_Code  : UINT;     (* Node error code *)
            Health      : USINT;    (* Node health status *)
        END_STRUCT
    END_TYPE
    
    (* State machine states *)
    TYPE DiagState :
        (IDLE, REQUEST_SENT, AWAITING_RESPONSE, PROCESS_DATA, ERROR_STATE)
    END_TYPE
    
    (* Internal state and timing *)
    Current_State      : DiagState := IDLE;  (* Current state machine state *)
    Request_Timer      : TON;                (* Timer for response timeout *)
    Request_Start_Tick : UDINT;              (* Tick at request start *)
    Response_Received  : BOOL := FALSE;      (* TRUE: Response received *)
    Last_Diag_Data     : DiagData;           (* Last valid diagnostic data *)
    
    (* Audit log *)
    Last_Log           : STRING[80];         (* Last logged message *)
    
    (* Error handling *)
    Timeout_Occurred   : BOOL := FALSE;      (* TRUE: Response timeout *)
END_VAR

(* Internal Methods *)
METHOD PRIVATE ValidateInputs : BOOL
    (* Validate Node_Address and Enable state *)
    IF Node_Address < 1 OR Node_Address > 240 THEN
        Error := TRUE;
        Error_Desc := 'Invalid node address';
        Audit_Log_Entry := 'Error: Invalid node address';
        RETURN FALSE;
    END_IF;
    IF NOT Enable THEN
        Error := TRUE;
        Error_Desc := 'Function block disabled';
        Audit_Log_Entry := 'Error: Function block disabled';
        RETURN FALSE;
    END_IF;
    RETURN TRUE;
END_METHOD

METHOD PRIVATE SimulateEPLResponse : DiagData
    VAR_INPUT
        NodeAddr : USINT;
    END_VAR
    (* Simulate EPL diagnostic response - replace with actual EPL API call *)
    (* In practice: CALL EPL_GetDiagData(NodeAddr, &diag); *)
    VAR
        diag : DiagData;
    END_VAR
    diag.Comm_Active := TRUE;  (* Simulated: Node is active *)
    diag.Error_Code := 0;      (* Simulated: No error *)
    diag.Health := 0;          (* Simulated: OK *)
    
    (* Simulate occasional errors *)
    IF NodeAddr = 10 AND (System_Tick_ms / 1000) MOD 20 = 0 THEN
        diag.Comm_Active := FALSE;
        diag.Error_Code := 1;  (* Simulated: Connection lost *)
        diag.Health := 2;      (* Simulated: Critical *)
    END_IF;
    
    RETURN diag;
END_METHOD

METHOD PRIVATE LogAuditEntry
    VAR_INPUT
        Message : STRING[80];
    END_VAR
    (* Log audit entry *)
    Audit_Log_Entry := Message;
    Last_Log := Message;
    (* In practice, export to HMI or logger: e.g., CALL LogToHMI(Audit_Log_Entry); *)
END_METHOD

METHOD PRIVATE GetErrorDescription : STRING[50]
    VAR_INPUT
        ErrCode : UINT;
    END_VAR
    (* Map error codes to descriptions *)
    CASE ErrCode OF
        0: RETURN 'No error';
        1: RETURN 'Connection lost';
        2: RETURN 'Timeout';
        3: RETURN 'Invalid response';
        ELSE RETURN 'Unknown error';
    END_CASE;
END_METHOD

(* Main Execution Logic *)
(* Reset outputs *)
Comm_Status := FALSE;
Error_Code := 0;
Node_Health := 0;
Diag_Valid := FALSE;
Error := FALSE;
Error_Desc := '';
Audit_Log_Entry := '';

(* State machine execution *)
CASE Current_State OF
    IDLE:
        (* Wait for request trigger or cyclic operation *)
        IF Enable AND R_EDGE(Request_Trigger) THEN
            IF ValidateInputs() THEN
                Current_State := REQUEST_SENT;
                Request_Start_Tick := System_Tick_ms;
                Request_Timer(IN := TRUE, PT := T#2000ms);  (* 2-second timeout *)
                LogAuditEntry(CONCAT('Request diagnostic data for node ', UINT_TO_STRING(Node_Address)));
                (* Simulate sending request *)
                (* In practice: CALL EPL_SendDiagRequest(Node_Address); *)
            ELSE
                Current_State := ERROR_STATE;
            END_IF;
        END_IF;
    
    REQUEST_SENT:
        (* Wait for response or timeout *)
        Request_Timer();
        IF Request_Timer.Q THEN
            Timeout_Occurred := TRUE;
            Current_State := ERROR_STATE;
            Error := TRUE;
            Error_Desc := 'Response timeout';
            Error_Code := 2;
            LogAuditEntry('Error: Response timeout for node ' + UINT_TO_STRING(Node_Address));
        ELSIF Response_Received THEN
            (* Simulated response - replace with actual EPL stack check *)
            Current_State := PROCESS_DATA;
            Request_Timer(IN := FALSE);
        END_IF;
        
        (* Simulate response for demo *)
        IF (System_Tick_ms - Request_Start_Tick) >= 500 THEN  (* Simulated 500ms delay *)
            Response_Received := TRUE;
            Last_Diag_Data := SimulateEPLResponse(Node_Address);
        END_IF;
    
    PROCESS_DATA:
        (* Process diagnostic data *)
        Comm_Status := Last_Diag_Data.Comm_Active;
        Error_Code := Last_Diag_Data.Error_Code;
        Node_Health := Last_Diag_Data.Health;
        Diag_Valid := TRUE;
        
        IF NOT Comm_Status OR Error_Code <> 0 THEN
            Error := TRUE;
            Error_Desc := GetErrorDescription(Error_Code);
            LogAuditEntry(CONCAT('Node ', UINT_TO_STRING(Node_Address), ': ', Error_Desc));
        ELSE
            LogAuditEntry(CONCAT('Node ', UINT_TO_STRING(Node_Address), ': Healthy'));
        END_IF;
        
        Current_State := IDLE;
        Response_Received := FALSE;
        Timeout_Occurred := FALSE;
    
    ERROR_STATE:
        (* Handle errors *)
        Diag_Valid := FALSE;
        IF Enable AND R_EDGE(Request_Trigger) THEN
            Current_State := IDLE;  (* Retry on new request *)
        END_IF;
END_CASE;

(* Timeout safety check *)
IF Current_State = REQUEST_SENT AND (System_Tick_ms - Request_Start_Tick) > 3000 THEN
    Current_State := ERROR_STATE;
    Error := TRUE;
    Error_Desc := 'Extended timeout';
    Error_Code := 2;
    LogAuditEntry('Error: Extended timeout for node ' + UINT_TO_STRING(Node_Address));
    Request così Request_Timer(IN := FALSE);
END_IF;

END_FUNCTION_BLOCK
