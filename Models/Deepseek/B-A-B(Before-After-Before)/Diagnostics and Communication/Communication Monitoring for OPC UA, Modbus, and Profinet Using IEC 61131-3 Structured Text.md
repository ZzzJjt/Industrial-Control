FUNCTION_BLOCK FB_CommMonitor
VAR_INPUT
    // Enable monitoring for each protocol
    EnableOPCUA : BOOL := TRUE;
    EnableModbus : BOOL := TRUE;
    EnableProfinet : BOOL := TRUE;
    
    // Configuration parameters
    PollingInterval : TIME := T#1S;  // Cyclic polling interval
    MaxTimeoutCount : UINT := 3;     // Consecutive timeouts before failure
END_VAR

VAR_OUTPUT
    // Status outputs
    OPCUA_Healthy : BOOL := FALSE;
    Modbus_Healthy : BOOL := FALSE;
    Profinet_Healthy : BOOL := FALSE;
    
    // Alarm outputs
    AlarmActive : BOOL := FALSE;
    AlarmProtocol : STRING(15) := '';
    AlarmReason : STRING(50) := '';
    AlarmCode : UINT := 0;
    
    // Audit trail interface
    AuditTrailWrite : BOOL := FALSE;
    AuditTrailEntry : STRING(100) := '';
END_VAR

VAR
    // Internal state tracking
    LastPollTime : TIME;
    PollTimer : TON;
    
    // Protocol state machines
    OPCUA_State : (IDLE, CONNECTING, CONNECTED, ERROR);
    Modbus_State : (IDLE, CONNECTING, CONNECTED, ERROR);
    Profinet_State : (IDLE, CONNECTING, CONNECTED, ERROR);
    
    // Error counters
    OPCUA_TimeoutCount : UINT := 0;
    Modbus_TimeoutCount : UINT := 0;
    Profinet_TimeoutCount : UINT := 0;
    
    // Previous states for edge detection
    PrevOPCUA_Healthy : BOOL := FALSE;
    PrevModbus_Healthy : BOOL := FALSE;
    PrevProfinet_Healthy : BOOL := FALSE;
    
    // Diagnostic buffers
    OPCUA_Diag : UDINT;
    Modbus_Diag : UDINT;
    Profinet_Diag : UDINT;
END_VAR

METHOD CheckOPCUAConnection : BOOL
VAR
    status : UINT;
    diagInfo : UDINT;
END_VAR
BEGIN
    // Call OPC UA client status function
    status := OPCUA_GetConnectionStatus(diagInfo);
    
    // Update diagnostic information
    OPCUA_Diag := diagInfo;
    
    // Evaluate connection state
    IF status = 16#0000 THEN
        OPCUA_TimeoutCount := 0;
        RETURN TRUE;
    ELSE
        OPCUA_TimeoutCount := OPCUA_TimeoutCount + 1;
        RETURN FALSE;
    END_IF;
END_METHOD

METHOD CheckModbusConnection : BOOL
VAR
    status : UINT;
    diagInfo : UDINT;
END_VAR
BEGIN
    // Call Modbus client status function
    status := Modbus_GetConnectionStatus(diagInfo);
    
    // Update diagnostic information
    Modbus_Diag := diagInfo;
    
    // Evaluate connection state
    IF status = 16#0000 THEN
        Modbus_TimeoutCount := 0;
        RETURN TRUE;
    ELSE
        Modbus_TimeoutCount := Modbus_TimeoutCount + 1;
        RETURN FALSE;
    END_IF;
END_METHOD

METHOD CheckProfinetConnection : BOOL
VAR
    status : UINT;
    diagInfo : UDINT;
END_VAR
BEGIN
    // Call Profinet device status function
    status := Profinet_GetDeviceStatus(diagInfo);
    
    // Update diagnostic information
    Profinet_Diag := diagInfo;
    
    // Evaluate connection state
    IF status = 16#0000 THEN
        Profinet_TimeoutCount := 0;
        RETURN TRUE;
    ELSE
        Profinet_TimeoutCount := Profinet_TimeoutCount + 1;
        RETURN FALSE;
    END_IF;
END_METHOD

METHOD LogAuditEvent
VAR_INPUT
    protocol : STRING(15);
    reason : STRING(50);
    code : UINT;
END_VAR
VAR
    timestamp : STRING(23);
END_VAR
BEGIN
    // Generate ISO 8601 timestamp
    timestamp := TIME_TO_STRING(DT_TO_TIME(DATE_AND_TIME()), 'hh:mm:ss');
    
    // Format audit trail entry
    AuditTrailEntry := CONCAT(CONCAT(CONCAT(CONCAT(
        '[', timestamp, '] '),
        protocol, ' - '),
        reason, ' (Code: 16#'),
        UINT_TO_HEX(code, 4), ')');
    
    // Trigger write pulse
    AuditTrailWrite := TRUE;
END_METHOD

// Main cyclic execution
BEGIN
    // Reset alarm outputs
    AlarmActive := FALSE;
    AuditTrailWrite := FALSE;
    
    // Cyclic polling using timer
    PollTimer(IN := TRUE, PT := PollingInterval);
    IF PollTimer.Q THEN
        PollTimer(IN := FALSE);
        
        // Check OPC UA connection if enabled
        IF EnableOPCUA THEN
            OPCUA_Healthy := CheckOPCUAConnection();
            
            // Detect state changes
            IF PrevOPCUA_Healthy AND NOT OPCUA_Healthy THEN
                // Connection lost
                AlarmActive := TRUE;
                AlarmProtocol := 'OPC UA';
                AlarmReason := 'Connection lost';
                AlarmCode := OPCUA_Diag;
                
                // Log to audit trail
                LogAuditEvent('OPC UA', 'Connection lost', OPCUA_Diag);
                
                // Update state machine
                OPCUA_State := ERROR;
            ELSIF NOT PrevOPCUA_Healthy AND OPCUA_Healthy THEN
                // Connection restored
                LogAuditEvent('OPC UA', 'Connection restored', 16#0000);
                OPCUA_State := CONNECTED;
            END_IF;
            
            PrevOPCUA_Healthy := OPCUA_Healthy;
        END_IF;
        
        // Check Modbus connection if enabled
        IF EnableModbus THEN
            Modbus_Healthy := CheckModbusConnection();
            
            // Detect state changes
            IF PrevModbus_Healthy AND NOT Modbus_Healthy THEN
                // Connection lost
                AlarmActive := TRUE;
                AlarmProtocol := 'Modbus';
                AlarmReason := 'Communication timeout';
                AlarmCode := Modbus_Diag;
                
                // Log to audit trail
                LogAuditEvent('Modbus', 'Communication timeout', Modbus_Diag);
                
                // Update state machine
                Modbus_State := ERROR;
            ELSIF NOT PrevModbus_Healthy AND Modbus_Healthy THEN
                // Connection restored
                LogAuditEvent('Modbus', 'Connection restored', 16#0000);
                Modbus_State := CONNECTED;
            END_IF;
            
            PrevModbus_Healthy := Modbus_Healthy;
        END_IF;
        
        // Check Profinet connection if enabled
        IF EnableProfinet THEN
            Profinet_Healthy := CheckProfinetConnection();
            
            // Detect state changes
            IF PrevProfinet_Healthy AND NOT Profinet_Healthy THEN
                // Connection lost
                AlarmActive := TRUE;
                AlarmProtocol := 'Profinet';
                AlarmReason := 'Device not responding';
                AlarmCode := Profinet_Diag;
                
                // Log to audit trail
                LogAuditEvent('Profinet', 'Device not responding', Profinet_Diag);
                
                // Update state machine
                Profinet_State := ERROR;
            ELSIF NOT PrevProfinet_Healthy AND Profinet_Healthy THEN
                // Connection restored
                LogAuditEvent('Profinet', 'Connection restored', 16#0000);
                Profinet_State := CONNECTED;
            END_IF;
            
            PrevProfinet_Healthy := Profinet_Healthy;
        END_IF;
        
        // Reset timer for next cycle
        PollTimer(IN := TRUE);
    END_IF;
END_FUNCTION_BLOCK
