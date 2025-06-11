FUNCTION_BLOCK FB_CommLinkMonitor
VAR_INPUT
    OPC_UA_CONNECTED: BOOL := FALSE;
    MODBUS_CONNECTED: BOOL := FALSE;
    PROFINET_CONNECTED: BOOL := FALSE;

    OPC_UA_ERROR_CODE: UINT := 0;
    MODBUS_ERROR_CODE: UINT := 0;
    PROFINET_ERROR_CODE: UINT := 0;

    OPC_UA_ERROR_MSG: STRING(80) := '';
    MODBUS_ERROR_MSG: STRING(80) := '';
    PROFINET_ERROR_MSG: STRING(80) := '';

    RESET_ALARM: BOOL := FALSE; // Optional reset input for manual alarm acknowledgment
END_VAR

VAR_OUTPUT
    ALM_OPC_UA_FAILURE: BOOL := FALSE;
    ALM_MODBUS_FAILURE: BOOL := FALSE;
    ALM_PROFINET_FAILURE: BOOL := FALSE;

    AUDIT_ENTRY_AVAILABLE: BOOL := FALSE;
END_VAR

VAR
    prev_opc_ua_state: BOOL := FALSE;
    prev_modbus_state: BOOL := FALSE;
    prev_profinet_state: BOOL := FALSE;

    last_opc_ua_time: TIME := T#0ms;
    last_modbus_time: TIME := T#0ms;
    last_profinet_time: TIME := T#0ms;

    audit_msg: STRING(255) := '';
END_VAR

VAR_IN_OUT
    pAuditBuffer: REF_TO ARRAY [0..9] OF STRING(255); // Reference to external audit buffer
    audit_index: INT := 0; // Current index in the buffer
END_VAR

FUNCTION_BLOCK FB_CommLinkMonitor
VAR_INPUT
    OPC_UA_CONNECTED: BOOL := FALSE;
    MODBUS_CONNECTED: BOOL := FALSE;
    PROFINET_CONNECTED: BOOL := FALSE;

    OPC_UA_ERROR_CODE: UINT := 0;
    MODBUS_ERROR_CODE: UINT := 0;
    PROFINET_ERROR_CODE: UINT := 0;

    OPC_UA_ERROR_MSG: STRING(80) := '';
    MODBUS_ERROR_MSG: STRING(80) := '';
    PROFINET_ERROR_MSG: STRING(80) := '';

    RESET_ALARM: BOOL := FALSE; // Optional reset input for manual alarm acknowledgment
END_VAR

VAR_OUTPUT
    ALM_OPC_UA_FAILURE: BOOL := FALSE;
    ALM_MODBUS_FAILURE: BOOL := FALSE;
    ALM_PROFINET_FAILURE: BOOL := FALSE;

    AUDIT_ENTRY_AVAILABLE: BOOL := FALSE;
END_VAR

VAR
    prev_opc_ua_state: BOOL := FALSE;
    prev_modbus_state: BOOL := FALSE;
    prev_profinet_state: BOOL := FALSE;

    last_opc_ua_time: TIME := T#0ms;
    last_modbus_time: TIME := T#0ms;
    last_profinet_time: TIME := T#0ms;

    audit_msg: STRING(255) := '';
END_VAR

VAR_IN_OUT
    pAuditBuffer: REF_TO ARRAY [0..9] OF STRING(255); // Reference to external audit buffer
    audit_index: INT := 0; // Current index in the buffer
END_VAR

METHOD PRIVATE LogAuditEntry
VAR_INPUT
    buffer: REF_TO ARRAY [0..*] OF STRING(255);
    msg: REF_TO STRING(255);
    index: INT;
END_VAR
VAR
    max_entries: INT := 10;
END_VAR

buffer[index] := msg^;
index := (index + 1) MOD max_entries;

PROGRAM PLC_PRG
VAR
    commMon: FB_CommLinkMonitor;
    auditLog: ARRAY [0..9] OF STRING(255);
    auditIndex: INT := 0;
    
    opcConnected: BOOL := TRUE;
    modbusConnected: BOOL := TRUE;
    profinetConnected: BOOL := FALSE;
    
    opcError: UINT := 102;
    modbusError: UINT := 5;
    profinetError: UINT := 201;
    
    opcMsg: STRING(80) := 'Connection Timeout';
    modbusMsg: STRING(80) := 'CRC Mismatch';
    profinetMsg: STRING(80) := 'Device Not Responding';
END_VAR

commMon(
    OPC_UA_CONNECTED := opcConnected,
    MODBUS_CONNECTED := modbusConnected,
    PROFINET_CONNECTED := profinetConnected,

    OPC_UA_ERROR_CODE := opcError,
    MODBUS_ERROR_CODE := modbusError,
    PROFINET_ERROR_CODE := profinetError,

    OPC_UA_ERROR_MSG := opcMsg,
    MODBUS_ERROR_MSG := modbusMsg,
    PROFINET_ERROR_MSG := profinetMsg,

    RESET_ALARM := FALSE,

    pAuditBuffer := ADR(auditLog),
    audit_index := auditIndex
);
