FUNCTION_BLOCK MODBUS_TCP_SERVER
(*
    Function Block: MODBUS_TCP_SERVER
    Purpose: Implements a Modbus TCP server supporting up to 10 concurrent client connections,
             handling function codes 0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x0F, 0x10, 0x17.
             Fully implements ReadCoils (0x01) with error handling and diagnostics.
    Standard: IEC 61131-3 Structured Text
    Date: May 20, 2025
*)

(* Input Variables *)
VAR_INPUT
    Enable           : BOOL;      (* TRUE: Enable server *)
    System_Tick_ms   : UDINT;     (* System tick in milliseconds for timing *)
    TCP_Receive      : BOOL;      (* TRUE: New data received on TCP socket *)
    TCP_Socket_ID    : UINT;      (* Socket ID for incoming data *)
    TCP_Data_In      : ARRAY[0..255] OF BYTE; (* Incoming TCP data *)
    TCP_Data_Len     : UINT;      (* Length of incoming data *)
END_VAR

(* Output Variables *)
VAR_OUTPUT
    TCP_Send         : BOOL;      (* TRUE: Ready to send response *)
    TCP_Data_Out     : ARRAY[0..255] OF BYTE; (* Outgoing TCP data *)
    TCP_Data_Out_Len : UINT;      (* Length of outgoing data *)
    Active_Clients   : UINT;      (* Number of active client connections *)
    Error            : BOOL;      (* TRUE: Error occurred *)
    Error_Desc       : STRING[50]; (* Error description *)
    Audit_Log_Entry  : STRING[80]; (* Audit log message *)
END_VAR

(* Internal Variables *)
VAR
    (* Modbus memory *)
    Coil_Memory      : ARRAY[0..9999] OF BOOL;    (* 10000 coils *)
    Discrete_Inputs  : ARRAY[0..9999] OF BOOL;    (* 10000 discrete inputs *)
    Holding_Registers: ARRAY[0..9999] OF UINT;    (* 10000 holding registers *)
    Input_Registers  : ARRAY[0..9999] OF UINT;    (* 10000 input registers *)
    
    (* Client session structure *)
    TYPE ClientSession :
        STRUCT
            Socket_ID    : UINT;      (* TCP socket ID *)
            Active       : BOOL;      (* TRUE: Session active *)
            Last_Activity: UDINT;     (* Last activity timestamp (ms) *)
            Trans_ID     : UINT;      (* Transaction identifier *)
            Timeout_Timer: TON;       (* Timeout timer *)
        END_STRUCT
    END_TYPE
    
    (* Session management *)
    Client_Sessions  : ARRAY[0..9] OF ClientSession; (* 10 client sessions *)
    Session_Initialized : BOOL := FALSE;             (* Session init flag *)
    
    (* Processing variables *)
    Current_Session  : UINT;      (* Index of current session *)
    PDU              : ARRAY[0..251] OF BYTE; (* Modbus PDU *)
    PDU_Len          : UINT;      (* PDU length *)
    MBAP_Header      : ARRAY[0..6] OF BYTE; (* MBAP header *)
    
    (* Audit log *)
    Last_Log         : STRING[80]; (* Last logged message *)
END_VAR

(* Internal Methods *)
METHOD PRIVATE InitializeSessions
    (* Initialize client sessions *)
    FOR i := 0 TO 9 DO
        Client_Sessions[i].Socket_ID := 0;
        Client_Sessions[i].Active := FALSE;
        Client_Sessions[i].Last_Activity := 0;
        Client_Sessions[i].Trans_ID := 0;
        Client_Sessions[i].Timeout_Timer(IN := FALSE, PT := T#5000ms); (* 5s timeout *)
    END_FOR;
    Session_Initialized := TRUE;
    Active_Clients := 0;
END_METHOD

METHOD PRIVATE FindSession : BOOL
    VAR_INPUT
        SocketID : UINT;
    END_VAR
    VAR_OUTPUT
        SessionIdx : UINT;
    END_VAR
    (* Find session by socket ID or allocate new session *)
    FOR i := 0 TO 9 DO
        IF Client_Sessions[i].Active AND Client_Sessions[i].Socket_ID = SocketID THEN
            SessionIdx := i;
            RETURN TRUE;
        END_IF;
    END_FOR;
    
    (* Allocate new session if available *)
    IF Active_Clients < 10 THEN
        FOR i := 0 TO 9 DO
            IF NOT Client_Sessions[i].Active THEN
                Client_Sessions[i].Socket_ID := SocketID;
                Client_Sessions[i].Active := TRUE;
                Client_Sessions[i].Last_Activity := System_Tick_ms;
                Client_Sessions[i].Timeout_Timer(IN := TRUE);
                SessionIdx := i;
                Active_Clients := Active_Clients + 1;
                RETURN TRUE;
            END_IF;
        END_FOR;
    END_IF;
    
    RETURN FALSE;
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

METHOD PRIVATE ReadCoils : BOOL
    VAR_INPUT
        StartAddr : UINT;
        Quantity  : UINT;
    END_VAR
    (* Implement Modbus function code 0x01: Read Coils *)
    (* Validate request *)
    IF Quantity < 1 OR Quantity > 2000 THEN
        PDU[0] := 16#81;  (* Exception: Function code + 0x80 *)
        PDU[1] := 16#03;  (* Illegal data value *)
        PDU_Len := 2;
        LogAuditEntry('ReadCoils: Invalid quantity');
        RETURN FALSE;
    END_IF;
    
    IF StartAddr + Quantity > 10000 THEN
        PDU[0] := 16#81;
        PDU[1] := 16#02;  (* Illegal data address *)
        PDU_Len := 2;
        LogAuditEntry('ReadCoils: Invalid address range');
        RETURN FALSE;
    END_IF;
    
    (* Calculate byte count *)
    VAR
        ByteCount : UINT := (Quantity + 7) / 8;
        CoilData  : ARRAY[0..249] OF BYTE;
        i, j      : UINT;
        CoilIdx   : UINT;
    END_VAR;
    
    (* Pack coil states into bytes *)
    FOR i := 0 TO ByteCount - 1 DO
        CoilData[i] := 0;
        FOR j := 0 TO 7 DO
            CoilIdx := StartAddr + i * 8 + j;
            IF CoilIdx < StartAddr + Quantity AND CoilIdx < 10000 THEN
                IF Coil_Memory[CoilIdx] THEN
                    CoilData[i] := CoilData[i] OR (1 SHL j);
                END_IF;
            END_IF;
        END_FOR;
    END_FOR;
    
    (* Build response PDU *)
    PDU[0] := 16#01;  (* Function code *)
    PDU[1] := ByteCount;
    FOR i := 0 TO ByteCount - 1 DO
        PDU[i + 2] := CoilData[i];
    END_FOR;
    PDU_Len := ByteCount + 2;
    
    LogAuditEntry(CONCAT('ReadCoils: Read ', UINT_TO_STRING(Quantity), ' coils from address ',
                         UINT_TO_STRING(StartAddr)));
    RETURN TRUE;
END_METHOD

METHOD PRIVATE ProcessPDU : BOOL
    (* Process Modbus PDU based on function code *)
    VAR
        FuncCode   : BYTE;
        StartAddr  : UINT;
        Quantity   : UINT;
    END_VAR
    FuncCode := PDU[0];
    
    CASE FuncCode OF
        16#01: (* Read Coils *)
            StartAddr := (PDU[1] SHL 8) OR PDU[2];
            Quantity := (PDU[3] SHL 8) OR PDU[4];
            RETURN ReadCoils(StartAddr, Quantity);
        
        16#02, 16#03, 16#04, 16#05, 16#06, 16#0F, 16#10, 16#17:
            (* Placeholder for other function codes *)
            PDU[0] := FuncCode OR 16#80;
            PDU[1] := 16#01;  (* Illegal function *)
            PDU_Len := 2;
            LogAuditEntry(CONCAT('Unsupported function code: ', BYTE_TO_STRING(FuncCode)));
            RETURN FALSE;
        
        ELSE
            PDU[0] := FuncCode OR 16#80;
            PDU[1] := 16#01;  (* Illegal function *)
            PDU_Len := 2;
            LogAuditEntry(CONCAT('Invalid function code: ', BYTE_TO_STRING(FuncCode)));
            RETURN FALSE;
    END_CASE;
END_METHOD

(* Main Execution Logic *)
(* Initialize sessions *)
IF NOT Session_Initialized THEN
    InitializeSessions();
END_IF;

(* Reset outputs *)
TCP_Send := FALSE;
TCP_Data_Out_Len := 0;
Error := FALSE;
Error_Desc := '';
Audit_Log_Entry := '';

(* Check if enabled *)
IF NOT Enable THEN
    Error := TRUE;
    Error_Desc := 'Server disabled';
    LogAuditEntry('Error: Server disabled');
    RETURN;
END_IF;

(* Manage client timeouts *)
FOR i := 0 TO 9 DO
    IF Client_Sessions[i].Active THEN
        Client_Sessions[i].Timeout_Timer();
        IF Client_Sessions[i].Timeout_Timer.Q THEN
            Client_Sessions[i].Active := FALSE;
            Client_Sessions[i].Socket_ID := 0;
            Active_Clients := Active_Clients - 1;
            LogAuditEntry(CONCAT('Client ', UINT_TO_STRING(i), ' timed out'));
        END_IF;
    END_IF;
END_FOR;

(* Process incoming data *)
IF TCP_Receive THEN
    (* Find or allocate session *)
    IF NOT FindSession(TCP_Socket_ID, Current_Session) THEN
        Error := TRUE;
        Error_Desc := 'No available sessions';
        LogAuditEntry('Error: No available sessions');
        RETURN;
    END_IF;
    
    (* Update session *)
    Client_Sessions[Current_Session].Last_Activity := System_Tick_ms;
    Client_Sessions[Current_Session].Timeout_Timer(IN := TRUE);
    
    (* Parse MBAP header *)
    IF TCP_Data_Len < 7 THEN
        Error := TRUE;
        Error_Desc := 'Invalid MBAP header';
        LogAuditEntry('Error: Invalid MBAP header');
        RETURN;
    END_IF;
    
    FOR i := 0 TO 6 DO
        MBAP_Header[i] := TCP_Data_In[i];
    END_FOR;
    Client_Sessions[Current_Session].Trans_ID := (MBAP_Header[0] SHL 8) OR MBAP_Header[1];
    PDU_Len := TCP_Data_Len - 7;
    
    (* Extract PDU *)
    IF PDU_Len > 251 THEN
        Error := TRUE;
        Error_Desc := 'PDU too long';
        LogAuditEntry('Error: PDU too long');
        RETURN;
    END_IF;
    
    FOR i := 0 TO PDU_Len - 1 DO
        PDU[i] := TCP_Data_In[i + 7];
    END_FOR;
    
    (* Process PDU *)
    IF ProcessPDU() THEN
        (* Build response *)
        MBAP_Header[4] := (PDU_Len + 1) SHR 8;  (* Length: PDU + Unit ID *)
        MBAP_Header[5] := (PDU_Len + 1) AND 16#FF;
        TCP_Data_Out_Len := PDU_Len + 7;
        FOR i := 0 TO 6 DO
            TCP_Data_Out[i] := MBAP_Header[i];
        END_FOR;
        FOR i := 0 TO PDU_Len - 1 DO
            TCP_Data_Out[i + 7] := PDU[i];
        END_FOR;
        TCP_Send := TRUE;
    ELSE
        (* Send exception response *)
        TCP_Data_Out_Len := 9;
        FOR i := 0 TO 6 DO
            TCP_Data_Out[i] := MBAP_Header[i];
        END_FOR;
        TCP_Data_Out[4] := 0;
        TCP_Data_Out[5] := 3;  (* Length: Exception PDU + Unit ID *)
        TCP_Data_Out[7] := PDU[0];  (* Exception function code *)
        TCP_Data_Out[8] := PDU[1];  (* Error code *)
        TCP_Send := TRUE;
    END_IF;
END_IF;

END_FUNCTION_BLOCK
