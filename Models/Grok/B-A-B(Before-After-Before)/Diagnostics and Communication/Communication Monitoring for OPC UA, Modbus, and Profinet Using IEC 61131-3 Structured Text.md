(* Function Block: NETWORK_MONITOR
   Purpose: Monitors OPC UA, Modbus, and Profinet connections, triggers alarms on failures, and logs audit trail entries.
   Features:
   - Cyclically polls connection status for each protocol
   - Detects faults (e.g., timeouts, disconnections, protocol errors)
   - Triggers alarms with protocol-specific error codes
   - Logs audit trail entries with timestamp, protocol, failure reason, and error code
*)

FUNCTION_BLOCK NETWORK_MONITOR
VAR_INPUT
    ENABLE : BOOL;                (* Enables/disables monitoring *)
    POLL_INTERVAL : TIME;         [* Polling interval for status checks (e.g., T#1s) *]
END_VAR

VAR_OUTPUT
    OPC_UA_ALARM : BOOL;          (* TRUE if OPC UA connection fails *)
    MODBUS_ALARM : BOOL;          (* TRUE if Modbus connection fails *)
    PROFINET_ALARM : BOOL;        (* TRUE if Profinet connection fails *)
    OPC_UA_ERROR_CODE : UINT;     (* OPC UA error code: 0=None, 1=Timeout, 2=Disconnected, 3=Protocol Error *)
    MODBUS_ERROR_CODE : UINT;     (* Modbus error code: 0=None, 1=Timeout, 2=Disconnected, 3=Protocol Error *)
    PROFINET_ERROR_CODE : UINT;   (* Profinet error code: 0=None, 1=Timeout, 2=Disconnected, 3=Protocol Error *)
    AUDIT_LOG_UPDATED : BOOL;     (* TRUE when a new audit log entry is created *)
END_VAR

VAR
    LastPollTime : TIME;          (* Timestamp of last poll *)
    OpcUaStatus : UINT;           (* OPC UA connection status *)
    ModbusStatus : UINT;          (* Modbus connection status *)
    ProfinetStatus : UINT;        (* Profinet connection status *)
    AuditLog : ARRAY[0..99] OF STRING[80]; (* Audit trail buffer, max 100 entries *)
    AuditLogIndex : UINT;         (* Current audit log index *)
    Timer : TON;                  (* Timer for polling interval *)
    LastOpcUaAlarm : BOOL;        (* Tracks previous OPC UA alarm state *)
    LastModbusAlarm : BOOL;       (* Tracks previous Modbus alarm state *)
    LastProfinetAlarm : BOOL;     (* Tracks previous Profinet alarm state *)
    CurrentTime : DT;             (* Current timestamp *)
END_VAR

(* Initialize outputs *)
OPC_UA_ALARM := FALSE;
MODBUS_ALARM := FALSE;
PROFINET_ALARM := FALSE;
OPC_UA_ERROR_CODE := 0;
MODBUS_ERROR_CODE := 0;
PROFINET_ERROR_CODE := 0;
AUDIT_LOG_UPDATED := FALSE;

(* Only execute if enabled *)
IF ENABLE THEN
    (* Update timer for polling *)
    Timer(IN := TRUE, PT := POLL_INTERVAL);
    
    (* Check if polling interval has elapsed *)
    IF Timer.Q THEN
        (* Reset timer *)
        Timer(IN := FALSE);
        
        (* Get current timestamp *)
        CurrentTime := GET_CURRENT_TIME();
        
        (* Poll OPC UA status *)
        OpcUaStatus := GET_OPC_UA_STATUS();
        IF OpcUaStatus <> 0 THEN
            OPC_UA_ALARM := TRUE;
            OPC_UA_ERROR_CODE := OpcUaStatus;
            IF NOT LastOpcUaAlarm THEN
                (* Log new failure *)
                AuditLog[AuditLogIndex] := CONCAT(
                    DT_TO_STRING(CurrentTime),
                    ': OPC UA Failure - ',
                    CASE OpcUaStatus OF
                        1: 'Timeout';
                        2: 'Disconnected';
                        3: 'Protocol Error';
                        ELSE: 'Unknown';
                    END_CASE,
                    ', Error Code: ',
                    UINT_TO_STRING(OpcUaStatus)
                );
                AuditLogIndex := AuditLogIndex + 1;
                IF AuditLogIndex >= 100 THEN
                    AuditLogIndex := 0; (* Circular buffer *)
                END_IF
                AUDIT_LOG_UPDATED := TRUE;
            END_IF
        ELSE
            OPC_UA_ALARM := FALSE;
            OPC_UA_ERROR_CODE := 0;
        END_IF
        LastOpcUaAlarm := OPC_UA_ALARM;
        
        (* Poll Modbus status *)
        ModbusStatus := GET_MODBUS_STATUS();
        IF ModbusStatus <> 0 THEN
            MODBUS_ALARM := TRUE;
            MODBUS_ERROR_CODE := ModbusStatus;
            IF NOT LastModbusAlarm THEN
                (* Log new failure *)
                AuditLog[AuditLogIndex] := CONCAT(
                    DT_TO_STRING(CurrentTime),
                    ': Modbus Failure - ',
                    CASE ModbusStatus OF
                        1: 'Timeout';
                        2: 'Disconnected';
                        3: 'Protocol Error';
                        ELSE: 'Unknown';
                    END_CASE,
                    ', Error Code: ',
                    UINT_TO_STRING(ModbusStatus)
                );
                AuditLogIndex := AuditLogIndex + 1;
                IF AuditLogIndex >= 100 THEN
                    AuditLogIndex := 0; (* Circular buffer *)
                END_IF
                AUDIT_LOG_UPDATED := TRUE;
            END_IF
        ELSE
            MODBUS_ALARM := FALSE;
            MODBUS_ERROR_CODE := 0;
        END_IF
        LastModbusAlarm := MODBUS_ALARM;
        
        (* Poll Profinet status *)
        ProfinetStatus := GET_PROFINET_STATUS();
        IF ProfinetStatus <> 0 THEN
            PROFINET_ALARM := TRUE;
            PROFINET_ERROR_CODE := ProfinetStatus;
            IF NOT LastProfinetAlarm THEN
                (* Log new failure *)
                AuditLog[AuditLogIndex] := CONCAT(
                    DT_TO_STRING(CurrentTime),
                    ': Profinet Failure - ',
                    CASE ProfinetStatus OF
                        1: 'Timeout';
                        2: 'Disconnected';
                        3: 'Protocol Error';
                        ELSE: 'Unknown';
                    END_CASE,
                    ', Error Code: ',
                    UINT_TO_STRING(ProfinetStatus)
                );
                AuditLogIndex := AuditLogIndex + 1;
                IF AuditLogIndex >= 100 THEN
                    AuditLogIndex := 0; (* Circular buffer *)
                END_IF
                AUDIT_LOG_UPDATED := TRUE;
            END_IF
        ELSE
            PROFINET_ALARM := FALSE;
            PROFINET_ERROR_CODE := 0;
        END_IF
        LastProfinetAlarm := PROFINET_ALARM;
        
        (* Reset AUDIT_LOG_UPDATED after one cycle *)
        AUDIT_LOG_UPDATED := FALSE;
    END_IF
ELSE
    (* Disable monitoring *)
    Timer(IN := FALSE);
    OPC_UA_ALARM := FALSE;
    MODBUS_ALARM := FALSE;
    PROFINET_ALARM := FALSE;
    OPC_UA_ERROR_CODE := 0;
    MODBUS_ERROR_CODE := 0;
    PROFINET_ERROR_CODE := 0;
    AUDIT_LOG_UPDATED := FALSE;
    LastOpcUaAlarm := FALSE;
    LastModbusAlarm := FALSE;
    LastProfinetAlarm := FALSE;
END_IF

(* Simulated protocol status functions *)
FUNCTION GET_OPC_UA_STATUS : UINT
(* Placeholder: Return 0 for OK, 1=Timeout, 2=Disconnected, 3=Protocol Error *)
GET_OPC_UA_STATUS := 0;
END_FUNCTION

FUNCTION GET_MODBUS_STATUS : UINT
(* Placeholder: Return 0 for OK, 1=Timeout, 2=Disconnected, 3=Protocol Error *)
GET_MODBUS_STATUS := 0;
END_FUNCTION

FUNCTION GET_PROFINET_STATUS : UINT
(* Placeholder: Return 0 for OK, 1=Timeout, 2=Disconnected, 3=Protocol Error *)
GET_PROFINET_STATUS := 0;
END_FUNCTION

FUNCTION GET_CURRENT_TIME : DT
(* Placeholder: Return current timestamp *)
GET_CURRENT_TIME := DT#2025-05-12-12:00:00;
END_FUNCTION

FUNCTION DT_TO_STRING : STRING
VAR_INPUT
    dt : DT;
END_VAR
(* Placeholder: Convert DT to string *)
DT_TO_STRING := '2025-05-12 12:00:00';
END_FUNCTION

FUNCTION UINT_TO_STRING : STRING
VAR_INPUT
    value : UINT;
END_VAR
(* Placeholder: Convert UINT to string *)
UINT_TO_STRING := '0';
END_FUNCTION
END_FUNCTION_BLOCK
