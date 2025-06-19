FUNCTION_BLOCK COMMUNICATION_MONITOR
VAR_INPUT
    OpcUaConnected : BOOL;      // Current connection state for OPC UA
    OpcUaErrorCode : INT;         // Error code for OPC UA
    ModbusConnected : BOOL;       // Current connection state for Modbus
    ModbusErrorCode : INT;        // Error code for Modbus
    ProfinetConnected : BOOL;     // Current connection state for Profinet
    ProfinetErrorCode : INT;      // Error code for Profinet
END_VAR

VAR_OUTPUT
    OpcUaAlarm : BOOL;           // Alarm flag for OPC UA
    ModbusAlarm : BOOL;          // Alarm flag for Modbus
    ProfinetAlarm : BOOL;        // Alarm flag for Profinet
END_VAR

VAR
    lastOpcUaConnected : BOOL := FALSE;
    lastModbusConnected : BOOL := FALSE;
    lastProfinetConnected : BOOL := FALSE;
    auditTrail : ARRAY[1..100] OF STRING; // Audit trail log
    auditIndex : INT := 0;               // Index for audit trail
    currentTime : TIME_OF_DAY;             // Current time of day
END_VAR

// Method to log an entry in the audit trail
METHOD LogAuditEntry : BOOL
VAR_INPUT
    protocolName : STRING;
    errorCode : INT;
    reason : STRING;
END_VAR
VAR
    logEntry : STRING;
END_VAR
    // Get current time
    currentTime := CURRENT_TIME_OF_DAY();

    // Format log entry
    logEntry := CONCAT(
        "Protocol: ", protocolName,
        ", Error Code: ", TO_STRING(errorCode),
        ", Reason: ", reason,
        ", Time: ", TO_STRING(currentTime)
    );

    // Add log entry to audit trail
    auditIndex := auditIndex + 1;
    IF auditIndex > SIZEOF(auditTrail) THEN
        auditIndex := 1; // Overwrite oldest entry if full
    END_IF;
    auditTrail[auditIndex] := logEntry;

    RETURN TRUE;
END_METHOD

// Main execution logic
METHOD Execute : BOOL
VAR
    opcUaReason, modbusReason, profinetReason : STRING;
END_VAR

    // Check OPC UA connection status
    IF NOT OpcUaConnected AND lastOpcUaConnected THEN
        OpcUaAlarm := TRUE;
        opcUaReason := "Connection lost";
        LogAuditEntry("OPC UA", OpcUaErrorCode, opcUaReason);
    ELSIF OpcUaConnected AND NOT lastOpcUaConnected THEN
        OpcUaAlarm := FALSE;
        opcUaReason := "Connection restored";
        LogAuditEntry("OPC UA", 0, opcUaReason);
    END_IF;

    // Check Modbus connection status
    IF NOT ModbusConnected AND lastModbusConnected THEN
        ModbusAlarm := TRUE;
        modbusReason := "Connection lost";
        LogAuditEntry("Modbus", ModbusErrorCode, modbusReason);
    ELSIF ModbusConnected AND NOT lastModbusConnected THEN
        ModbusAlarm := FALSE;
        modbusReason := "Connection restored";
        LogAuditEntry("Modbus", 0, modbusReason);
    END_IF;

    // Check Profinet connection status
    IF NOT ProfinetConnected AND lastProfinetConnected THEN
        ProfinetAlarm := TRUE;
        profinetReason := "Connection lost";
        LogAuditEntry("Profinet", ProfinetErrorCode, profinetReason);
    ELSIF ProfinetConnected AND NOT lastProfinetConnected THEN
        ProfinetAlarm := FALSE;
        profinetReason := "Connection restored";
        LogAuditEntry("Profinet", 0, profinetReason);
    END_IF;

    // Update last connection states
    lastOpcUaConnected := OpcUaConnected;
    lastModbusConnected := ModbusConnected;
    lastProfinetConnected := ProfinetConnected;

    RETURN TRUE;
END_METHOD

END_FUNCTION_BLOCK

