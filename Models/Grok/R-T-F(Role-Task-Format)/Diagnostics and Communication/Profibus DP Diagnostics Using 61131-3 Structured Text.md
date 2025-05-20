FUNCTION_BLOCK PROFIBUS_DIAG_READ
(*
    Function Block: PROFIBUS_DIAG_READ
    Purpose: Reads diagnostic data from a Profibus DP slave device, including device status,
             communication health, and error codes, with timeout and error handling.
    Standard: IEC 61131-3 Structured Text
    Date: May 20, 2025
*)

(* Input Variables *)
VAR_INPUT
    Execute          : BOOL;      (* R_EDGE: Trigger diagnostic read *)
    SlaveAddress     : BYTE;      (* Profibus slave address (1..125) *)
    Timeout          : TIME;      (* Max wait time for response *)
    System_Tick_ms   : UDINT;     (* System tick in milliseconds *)
END_VAR

(* Output Variables *)
VAR_OUTPUT
    Done             : BOOL;      (* TRUE: Diagnostics retrieved *)
    Busy             : BOOL;      (* TRUE: Operation in progress *)
    Error            : BOOL;      (* TRUE: Operation failed *)
    ErrorID          : DWORD;     (* Error code: 0x0000=No error, 0x0001=Invalid address,
                                     0x0002=Timeout, 0x0003=Bus fault, 0x0004=Slave error *)
    DeviceStatus     : BYTE;      (* Slave status: 0=OK, 1=Fault, 2=Config error *)
    CommHealth       : BYTE;      (* Comm health: 0=OK, 1=Warning, 2=Critical *)
    ExtendedDiag     : ARRAY[0..31] OF BYTE; (* Extended diagnostic data *)
    Audit_Log_Entry  : STRING[80]; (* Audit log message *)
END_VAR

(* Internal Variables *)
VAR
    (* State machine states *)
    TYPE DiagState :
        (IDLE, REQUEST_SENT, PROCESS_RESPONSE, ERROR_STATE)
    END_TYPE
    
    (* Diagnostic data structure *)
    TYPE DiagData :
        STRUCT
            Status      : BYTE;      (* Device status *)
            CommHealth  : BYTE;      (* Communication health *)
            DiagData    : ARRAY[0..31] OF BYTE; (* Extended diagnostics *)
            Valid       : BOOL;      (* TRUE: Data valid *)
            ErrorCode   : UINT;      (* Slave error code *)
        END_STRUCT
    END_TYPE
    
    Current_State    : DiagState := IDLE; (* Current state *)
    Prev_Execute     : BOOL := FALSE;     (* Previous Execute state *)
    Request_Timer    : TON;              (* Timeout timer *)
    Start_Tick       : UDINT;            (* Tick at request start *)
    Response_Received : BOOL := FALSE;   (* TRUE: Response received *)
    Last_Diag_Data   : DiagData;         (* Last diagnostic data *)
    Last_Log         : STRING[80];       (* Last logged message *)
END_VAR

(* Internal Methods *)
METHOD PRIVATE ValidateInputs : BOOL
    (* Validate inputs *)
    IF SlaveAddress < 1 OR SlaveAddress > 125 THEN
        Error := TRUE;
        ErrorID := 16#0001; (* Invalid address *)
        Audit_Log_Entry := 'Error: Invalid slave address';
        RETURN FALSE;
    END_IF;
    IF Timeout < T#100ms THEN
        Error := TRUE;
        ErrorID := 16#FFFF; (* Invalid timeout *)
        Audit_Log_Entry := 'Error: Invalid timeout value';
        RETURN FALSE;
    END_IF;
    RETURN TRUE;
END_METHOD

METHOD PRIVATE SimulateDPResponse : DiagData
    VAR_INPUT
        Addr : BYTE;
    END_VAR
    (* Simulate Profibus DP diagnostic response *)
    (* In practice: CALL DP_DIAG_REQUEST(Addr, &diag); *)
    VAR
        result : DiagData;
    END_VAR
    result.Status := 0;      (* OK *)
    result.CommHealth := 0; (* OK *)
    result.Valid := TRUE;
    result.ErrorCode := 0;
    FOR i := 0 TO 31 DO
        result.DiagData[i] := 0;
    END_FOR;
    
    (* Simulate errors *)
    IF Addr = 10 AND (System_Tick_ms / 1000) MOD 20 = 0 THEN
        result.Status := 1;      (* Fault *)
        result.CommHealth := 2;  (* Critical *)
        result.ErrorCode := 1;   (* Simulated: Slave fault *)
        result.DiagData[0] := 16#01; (* Example extended diag *)
    END_IF;
    
    RETURN result;
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
        ErrID : DWORD;
    END_VAR
    (* Map ErrorID to description *)
    CASE ErrID OF
        16#0000: RETURN 'Diagnostics retrieved';
        16#0001: RETURN 'Invalid slave address';
        16#0002: RETURN 'Timeout';
        16#0003: RETURN 'Bus fault';
        16#0004: RETURN 'Slave error';
        ELSE RETURN 'Unknown error';
    END_CASE;
END_METHOD

(* Main Execution Logic *)
(* Reset outputs *)
Done := FALSE;
Busy := FALSE;
Error := FALSE;
ErrorID := 16#0000;
DeviceStatus := 0;
CommHealth := 0;
FOR i := 0 TO 31 DO
    ExtendedDiag[i] := 0;
END_FOR;
Audit_Log_Entry := '';

(* Handle state machine *)
CASE Current_State OF
    IDLE:
        (* Wait for rising edge on Execute *)
        IF Execute AND NOT Prev_Execute THEN
            IF ValidateInputs() THEN
                Current_State := REQUEST_SENT;
                Busy := TRUE;
                Start_Tick := System_Tick_ms;
                Request_Timer(IN := TRUE, PT := Timeout);
                LogAuditEntry(CONCAT('Request diagnostics for slave ', BYTE_TO_STRING(SlaveAddress)));
                (* Simulate sending request *)
                (* In practice: CALL DP_DIAG_REQUEST(SlaveAddress, &diag_data); *)
            ELSE
                Current_State := ERROR_STATE;
            END_IF;
        END_IF;
    
    REQUEST_SENT:
        (* Wait for response or timeout *)
        Request_Timer();
        IF Request_Timer.Q THEN
            Current_State := ERROR_STATE;
            Error := TRUE;
            ErrorID := 16#0002; (* Timeout *)
            LogAuditEntry(CONCAT('Error: Timeout for slave ', BYTE_TO_STRING(SlaveAddress)));
        ELSIF Response_Received THEN
            Current_State := PROCESS_RESPONSE;
            Request_Timer(IN := FALSE);
        END_IF;
        
        (* Simulate response for demo *)
        IF (System_Tick_ms - Start_Tick) >= 200 THEN  (* Simulated 200ms delay *)
            Response_Received := TRUE;
            Last_Diag_Data := SimulateDPResponse(SlaveAddress);
        END_IF;
    
    PROCESS_RESPONSE:
        (* Process diagnostic data *)
        IF Last_Diag_Data.Valid THEN
            DeviceStatus := Last_Diag_Data.Status;
            CommHealth := Last_Diag_Data.CommHealth;
            FOR i := 0 TO 31 DO
                ExtendedDiag[i] := Last_Diag_Data.DiagData[i];
            END_FOR;
            Done := TRUE;
            IF Last_Diag_Data.ErrorCode <> 0 THEN
                Error := TRUE;
                ErrorID := 16#0004; (* Slave error *)
                LogAuditEntry(CONCAT('Slave ', BYTE_TO_STRING(SlaveAddress), ': Error code ',
                                     UINT_TO_STRING(Last_Diag_Data.ErrorCode)));
            ELSE
                LogAuditEntry(CONCAT('Diagnostics retrieved for slave ', BYTE_TO_STRING(SlaveAddress)));
            END_IF;
        ELSE
            Current_State := ERROR_STATE;
            Error := TRUE;
            ErrorID := 16#0003; (* Bus fault *)
            LogAuditEntry(CONCAT('Error: Invalid response for slave ', BYTE_TO_STRING(SlaveAddress)));
        END_IF;
        Current_State := IDLE;
        Response_Received := FALSE;
    
    ERROR_STATE:
        (* Handle error *)
        Done := FALSE;
        Busy := FALSE;
        Error := TRUE;
        LogAuditEntry(CONCAT('Error: ', GetErrorDescription(ErrorID), ' for slave ',
                             BYTE_TO_STRING(SlaveAddress)));
        IF NOT Execute THEN
            Current_State := IDLE; (* Reset on falling edge *)
        END_IF;
END_CASE;

(* Update previous Execute state *)
Prev_Execute := Execute;

(* Safety check for extended delay *)
IF Current_State = REQUEST_SENT AND (System_Tick_ms - Start_Tick) > TIME_TO_MS(Timeout) + 1000 THEN
    Current_State := ERROR_STATE;
    Error := TRUE;
    ErrorID := 16#0002; (* Timeout *)
    Request_Timer(IN := FALSE);
    LogAuditEntry(CONCAT('Error: Extended timeout for slave ', BYTE_TO_STRING(SlaveAddress)));
END_IF;

END_FUNCTION_BLOCK
