FUNCTION_BLOCK COMM_MONITOR
VAR_INPUT
    EXECUTE: BOOL;             // Cyclic execution trigger
    OPCUA_STATUS: BOOL;        // TRUE: OPC UA connected, FALSE: disconnected
    OPCUA_ERROR: INT;          // OPC UA error code (0: no error)
    MODBUS_STATUS: BOOL;       // TRUE: Modbus connected, FALSE: disconnected
    MODBUS_ERROR: INT;         // Modbus error code (0: no error)
    PROFINET_STATUS: BOOL;     // TRUE: Profinet connected, FALSE: disconnected
    PROFINET_ERROR: INT;       // Profinet error code (0: no error)
END_VAR

VAR_OUTPUT
    OPCUA_ALARM: BOOL;         // TRUE: OPC UA failure detected
    OPCUA_ERRCODE: INT;        // OPC UA error code
    MODBUS_ALARM: BOOL;        // TRUE: Modbus failure detected
    MODBUS_ERRCODE: INT;       // Modbus error code
    PROFINET_ALARM: BOOL;      // TRUE: Profinet failure detected
    PROFINET_ERRCODE: INT;     // Profinet error code
    AUDIT_COUNT: INT;          // Number of audit entries
    ERROR: BOOL;               // TRUE: Internal error
    ERROR_CODE: INT;           // 0: No error, 1: Audit buffer fault
END_VAR

VAR
    // Audit entry structure
    TYPE AUDIT_ENTRY:
        STRUCT
            TIMESTAMP: TIME;    // Time of event
            PROTOCOL: STRING[16]; // Protocol name
            REASON: STRING[32]; // Failure or recovery reason
            ERROR_CODE: INT;    // Associated error code
        END_STRUCT;
    END_TYPE
    
    AuditBuffer: ARRAY[1..100] OF AUDIT_ENTRY; // Circular audit buffer
    AuditHead: INT := 1;                       // Next write position
    AuditOverflow: BOOL;                       // TRUE if buffer overwrites
    OpcUaPrevStatus: BOOL;                     // Previous OPC UA status
    ModbusPrevStatus: BOOL;                    // Previous Modbus status
    ProfinetPrevStatus: BOOL;                  // Previous Profinet status
    ExecuteEdge: BOOL;                         // Edge detection for EXECUTE
END_VAR

// Reset outputs when not executing
IF NOT EXECUTE THEN
    OPCUA_ALARM := FALSE;
    OPCUA_ERRCODE := 0;
    MODBUS_ALARM := FALSE;
    MODBUS_ERRCODE := 0;
    PROFINET_ALARM := FALSE;
    PROFINET_ERRCODE := 0;
    ERROR := FALSE;
    ERROR_CODE := 0;
    ExecuteEdge := FALSE;
    RETURN;
END_IF;

// Cyclic execution on rising edge or sustained EXECUTE
IF EXECUTE THEN
    // Detect rising edge for initialization
    IF NOT ExecuteEdge THEN
        ExecuteEdge := TRUE;
        OpcUaPrevStatus := OPCUA_STATUS;
        ModbusPrevStatus := MODBUS_STATUS;
        ProfinetPrevStatus := PROFINET_STATUS;
    END_IF;
    
    // Reset error flags for this cycle
    ERROR := FALSE;
    ERROR_CODE := 0;
    
    // Validate audit buffer
    IF AuditHead < 1 OR AuditHead > 100 THEN
        ERROR := TRUE;
        ERROR_CODE := 1; // Audit buffer fault
        RETURN;
    END_IF;
    
    // Process OPC UA
    IF OPCUA_STATUS <> OpcUaPrevStatus THEN
        IF NOT OPCUA_STATUS THEN
            // Failure detected
            OPCUA_ALARM := TRUE;
            OPCUA_ERRCODE := OPCUA_ERROR;
            AuditBuffer[AuditHead].TIMESTAMP := TIME();
            AuditBuffer[AuditHead].PROTOCOL := 'OPC UA';
            AuditBuffer[AuditHead].REASON := 'Connection Lost';
            AuditBuffer[AuditHead].ERROR_CODE := OPCUA_ERROR;
            AuditHead := AuditHead + 1;
            IF AuditHead > 100 THEN
                AuditHead := 1;
                AuditOverflow := TRUE;
            END_IF;
            AUDIT_COUNT := MIN(AUDIT_COUNT + 1, 100);
        ELSE
            // Recovery detected
            OPCUA_ALARM := FALSE;
            OPCUA_ERRCODE := 0;
            AuditBuffer[AuditHead].TIMESTAMP := TIME();
            AuditBuffer[AuditHead].PROTOCOL := 'OPC UA';
            AuditBuffer[AuditHead].REASON := 'Connection Restored';
            AuditBuffer[AuditHead].ERROR_CODE := 0;
            AuditHead := AuditHead + 1;
            IF AuditHead > 100 THEN
                AuditHead := 1;
                AuditOverflow := TRUE;
            END_IF;
            AUDIT_COUNT := MIN(AUDIT_COUNT + 1, 100);
        END_IF;
    END_IF;
    OpcUaPrevStatus := OPCUA_STATUS;
    
    // Process Modbus
    IF MODBUS_STATUS <> ModbusPrevStatus THEN
        IF NOT MODBUS_STATUS THEN
            // Failure detected
            MODBUS_ALARM := TRUE;
            MODBUS_ERRCODE := MODBUS_ERROR;
            AuditBuffer[AuditHead].TIMESTAMP := TIME();
            AuditBuffer[AuditHead].PROTOCOL := 'Modbus';
            AuditBuffer[AuditHead].REASON := 'Connection Lost';
            AuditBuffer[AuditHead].ERROR_CODE := MODBUS_ERROR;
            AuditHead := AuditHead + 1;
            IF AuditHead > 100 THEN
                AuditHead := 1;
                AuditOverflow := TRUE;
            END_IF;
            AUDIT_COUNT := MIN(AUDIT_COUNT + 1, 100);
        ELSE
            // Recovery detected
            MODBUS_ALARM := FALSE;
            MODBUS_ERRCODE := 0;
            AuditBuffer[AuditHead].TIMESTAMP := TIME();
            AuditBuffer[AuditHead].PROTOCOL := 'Modbus';
            AuditBuffer[AuditHead].REASON := 'Connection Restored';
            AuditBuffer[AuditHead].ERROR_CODE := 0;
            AuditHead := AuditHead + 1;
            IF AuditHead > 100 THEN
                AuditHead := 1;
                AuditOverflow := TRUE;
            END_IF;
            AUDIT_COUNT := MIN(AUDIT_COUNT + 1, 100);
        END_IF;
    END_IF;
    ModbusPrevStatus := MODBUS_STATUS;
    
    // Process Profinet
    IF PROFINET_STATUS <> ProfinetPrevStatus THEN
        IF NOT PROFINET_STATUS THEN
            // Failure detected
            PROFINET_ALARM := TRUE;
            PROFINET_ERRCODE := PROFINET_ERROR;
            AuditBuffer[AuditHead].TIMESTAMP := TIME();
            AuditBuffer[AuditHead].PROTOCOL := 'Profinet';
            AuditBuffer[AuditHead].REASON := 'Connection Lost';
            AuditBuffer[AuditHead].ERROR_CODE := PROFINET_ERROR;
            AuditHead := AuditHead + 1;
            IF AuditHead > 100 THEN
                AuditHead := 1;
                AuditOverflow := TRUE;
            END_IF;
            AUDIT_COUNT := MIN(AUDIT_COUNT + 1, 100);
        ELSE
            // Recovery detected
            PROFINET_ALARM := FALSE;
            PROFINET_ERRCODE := 0;
            AuditBuffer[AuditHead].TIMESTAMP := TIME();
            AuditBuffer[AuditHead].PROTOCOL := 'Profinet';
            AuditBuffer[AuditHead].REASON := 'Connection Restored';
            AuditBuffer[AuditHead].ERROR_CODE := 0;
            AuditHead := AuditHead + 1;
            IF AuditHead > 100 THEN
                AuditHead := 1;
                AuditOverflow := TRUE;
            END_IF;
            AUDIT_COUNT := MIN(AUDIT_COUNT + 1, 100);
        END_IF;
    END_IF;
    ProfinetPrevStatus := PROFINET_STATUS;
    
    // Validate error codes
    IF OPCUA_ALARM AND OPCUA_ERRCODE = 0 THEN
        ERROR := TRUE;
        ERROR_CODE := 1; // Invalid error code
    ELSIF MODBUS_ALARM AND MODBUS_ERRCODE = 0 THEN
        ERROR := TRUE;
        ERROR_CODE := 1;
    ELSIF PROFINET_ALARM AND PROFINET_ERRCODE = 0 THEN
        ERROR := TRUE;
        ERROR_CODE := 1;
    END_IF;
END_IF;
END_FUNCTION_BLOCK
