FUNCTION_BLOCK COMM_MONITOR
(*
    Function Block: COMM_MONITOR
    Purpose: Monitors communication status of OPC UA, Modbus, and Profinet protocols,
             triggers alarms on failures, and logs audit trail entries.
    Standard: IEC 61131-3 Structured Text
*)

(* Input Variables *)
VAR_INPUT
    OPCUA_Connected   : BOOL;    (* TRUE if OPC UA connection is active *)
    OPCUA_ErrorCode   : UINT;    (* OPC UA error code, 0 = no error *)
    Modbus_Connected  : BOOL;    (* TRUE if Modbus connection is active *)
    Modbus_ErrorCode  : UINT;    (* Modbus error code, 0 = no error *)
    Profinet_Connected: BOOL;    (* TRUE if Profinet connection is active *)
    Profinet_ErrorCode: UINT;    (* Profinet error code, 0 = no error *)
    System_Timestamp  : DT;      (* Current system timestamp *)
END_VAR

(* Output Variables *)
VAR_OUTPUT
    OPCUA_Alarm       : BOOL;    (* TRUE if OPC UA communication failure detected *)
    Modbus_Alarm      : BOOL;    (* TRUE if Modbus communication failure detected *)
    Profinet_Alarm    : BOOL;    (* TRUE if Profinet communication failure detected *)
    Audit_Log_Updated : BOOL;    (* TRUE when a new audit log entry is added *)
    Audit_Log_Full    : BOOL;    (* TRUE if audit log buffer is full *)
END_VAR

(* Internal Variables *)
VAR
    (* Audit log structure *)
    TYPE AuditEntry :
        STRUCT
            Timestamp : DT;        (* Time of event *)
            Protocol  : STRING[10]; (* Protocol name: OPCUA, Modbus, Profinet *)
            Status    : STRING[20]; (* Status: Failure, Recovery *)
            ErrorCode : UINT;      (* Error code *)
            Reason    : STRING[50]; (* Error reason *)
        END_STRUCT
    END_TYPE
    
    (* Audit log buffer *)
    Audit_Log       : ARRAY[0..99] OF AuditEntry; (* Max 100 entries *)
    Log_Index       : UINT := 0;                  (* Current log index *)
    Log_Initialized : BOOL := FALSE;              (* Log initialization flag *)
    
    (* Previous connection states for edge detection *)
    OPCUA_Prev_Connected   : BOOL := TRUE;
    Modbus_Prev_Connected  : BOOL := TRUE;
    Profinet_Prev_Connected: BOOL := TRUE;
    
    (* Alarm persistence *)
    OPCUA_Alarm_Latched    : BOOL := FALSE;
    Modbus_Alarm_Latched   : BOOL := FALSE;
    Profinet_Alarm_Latched : BOOL := FALSE;
END_VAR

(* Internal Methods *)
METHOD PRIVATE InitializeLog
    (* Initialize audit log buffer *)
    FOR i := 0 TO 99 DO
        Audit_Log[i].Timestamp := DT#1970-01-01-00:00:00;
        Audit_Log[i].Protocol := '';
        Audit_Log[i].Status := '';
        Audit_Log[i].ErrorCode := 0;
        Audit_Log[i].Reason := '';
    END_FOR;
    Log_Index := 0;
    Log_Initialized := TRUE;
    Audit_Log_Full := FALSE;
END_METHOD

METHOD PRIVATE AddLogEntry
    VAR_INPUT
        Protocol  : STRING[10];
        Status    : STRING[20];
        ErrorCode : UINT;
        Reason    : STRING[50];
    END_VAR
    (* Add entry to audit log if buffer not full *)
    IF Log_Index < 100 THEN
        Audit_Log[Log_Index].Timestamp := System_Timestamp;
        Audit_Log[Log_Index].Protocol := Protocol;
        Audit_Log[Log_Index].Status := Status;
        Audit_Log[Log_Index].ErrorCode := ErrorCode;
        Audit_Log[Log_Index].Reason := Reason;
        Log_Index := Log_Index + 1;
        Audit_Log_Updated := TRUE;
    ELSE
        Audit_Log_Full := TRUE;
        Audit_Log_Updated := FALSE;
    END_IF;
END_METHOD

METHOD PRIVATE GetErrorReason : STRING[50]
    VAR_INPUT
        Protocol  : STRING[10];
        ErrorCode : UINT;
    END_VAR
    (* Map error codes to reasons - simplified for demo *)
    CASE Protocol OF
        'OPCUA':
            CASE ErrorCode OF
                1: RETURN 'Connection timeout';
                2: RETURN 'Authentication failure';
                ELSE RETURN 'Unknown OPC UA error';
            END_CASE;
        'Modbus':
            CASE ErrorCode OF
                1: RETURN 'CRC error';
                2: RETURN 'Timeout';
                ELSE RETURN 'Unknown Modbus error';
            END_CASE;
        'Profinet':
            CASE ErrorCode OF
                1: RETURN 'Network down';
                2: RETURN 'Device unreachable';
                ELSE RETURN 'Unknown Profinet error';
            END_CASE;
        ELSE
            RETURN 'Invalid protocol';
    END_CASE;
END_METHOD

(* Main Execution Logic *)
(* Initialize log if not done *)
IF NOT Log_Initialized THEN
    InitializeLog();
END_IF;

(* Reset outputs *)
OPCUA_Alarm := FALSE;
Modbus_Alarm := FALSE;
Profinet_Alarm := FALSE;
Audit_Log_Updated := FALSE;

(* Monitor OPC UA *)
IF OPCUA_Connected <> OPCUA_Prev_Connected THEN
    IF NOT OPCUA_Connected THEN
        (* Failure detected *)
        OPCUA_Alarm := TRUE;
        OPCUA_Alarm_Latched := TRUE;
        AddLogEntry('OPCUA', 'Failure', OPCUA_ErrorCode, GetErrorReason('OPCUA', OPCUA_ErrorCode));
    ELSE
        (* Recovery detected *)
        OPCUA_Alarm_Latched := FALSE;
        AddLogEntry('OPCUA', 'Recovery', 0, 'Connection restored');
    END_IF;
END_IF;
OPCUA_Prev_Connected := OPCUA_Connected;
OPCUA_Alarm := OPCUA_Alarm_Latched;

(* Monitor Modbus *)
IF Modbus_Connected <> Modbus_Prev_Connected THEN
    IF NOT Modbus_Connected THEN
        (* Failure detected *)
        Modbus_Alarm := TRUE;
        Modbus_Alarm_Latched := TRUE;
        AddLogEntry('Modbus', 'Failure', Modbus_ErrorCode, GetErrorReason('Modbus', Modbus_ErrorCode));
    ELSE
        (* Recovery detected *)
        Modbus_Alarm_Latched := FALSE;
        AddLogEntry('Modbus', 'Recovery', 0, 'Connection restored');
    END_IF;
END_IF;
Modbus_Prev_Connected := Modbus_Connected;
Modbus_Alarm := Modbus_Alarm_Latched;

(* Monitor Profinet *)
IF Profinet_Connected <> Profinet_Prev_Connected THEN
    IF NOT Profinet_Connected THEN
        (* Failure detected *)
        Profinet_Alarm := TRUE;
        Profinet_Alarm_Latched := TRUE;
        AddLogEntry('Profinet', 'Failure', Profinet_ErrorCode, GetErrorReason('Profinet', Profinet_ErrorCode));
    ELSE
        (* Recovery detected *)
        Profinet_Alarm_Latched := FALSE;
        AddLogEntry('Profinet', 'Recovery', 0, 'Connection restored');
    END_IF;
END_IF;
Profinet_Prev_Connected := Profinet_Connected;
Profinet_Alarm := Profinet_Alarm_Latched;

(* Log buffer status *)
IF Log_Index >= 100 THEN
    Audit_Log_Full := TRUE;
END_IF;

(* Note: Audit log can be exported to HMI or external system *)
(* e.g., CALL ExportAuditLog(Audit_Log, Log_Index); *)
END_FUNCTION_BLOCK
