FUNCTION_BLOCK COMMUNICATION_MONITOR
VAR_INPUT
    ENABLE : BOOL; // Enable monitoring
END_VAR

VAR_OUTPUT
    ALARM : BOOL; // Alarm signal if any connection fails
    AUDIT_TRAIL : ARRAY[1..10] OF STRING[100]; // Audit trail entries
    NUM_ENTRIES : INT; // Number of audit trail entries
END_VAR

VAR
    opcua_connected : BOOL := FALSE;
    modbus_connected : BOOL := FALSE;
    profinet_connected : BOOL := FALSE;
    last_opcua_status : BOOL := FALSE;
    last_modbus_status : BOOL := FALSE;
    last_profinet_status : BOOL := FALSE;
    max_audit_entries : INT := 10;
END_VAR

METHOD CheckOPCUAConnection : BOOL
BEGIN
    // Simulate checking OPC UA connection status
    // Replace with actual API calls in real application
    opcua_connected := TRUE; // Example: Assume connection is always true for demonstration
    RETURN opcua_connected;
END_METHOD

METHOD CheckModbusConnection : BOOL
BEGIN
    // Simulate checking Modbus connection status
    // Replace with actual API calls in real application
    modbus_connected := TRUE; // Example: Assume connection is always true for demonstration
    RETURN modbus_connected;
END_METHOD

METHOD CheckProfinetConnection : BOOL
BEGIN
    // Simulate checking Profinet connection status
    // Replace with actual API calls in real application
    profinet_connected := TRUE; // Example: Assume connection is always true for demonstration
    RETURN profinet_connected;
END_METHOD

METHOD LogAuditTrail : BOOL
VAR_INPUT
    protocol_name : STRING[20];
    reason : STRING[50];
    error_code : INT;
END_VAR
VAR
    log_entry : STRING[100];
BEGIN
    IF NUM_ENTRIES < max_audit_entries THEN
        NUM_ENTRIES := NUM_ENTRIES + 1;
        log_entry := CONCAT(protocol_name, ': ', reason, ' (Error Code: ', TO_STRING(error_code), ')');
        AUDIT_TRAIL[NUM_ENTRIES] := log_entry;
        RETURN TRUE;
    ELSE
        RETURN FALSE;
    END_IF;
END_METHOD

METHOD Execute : BOOL
VAR
    opcua_status : BOOL;
    modbus_status : BOOL;
    profinet_status : BOOL;
BEGIN
    IF NOT ENABLE THEN
        ALARM := FALSE;
        RETURN FALSE;
    END_IF;

    opcua_status := CheckOPCUAConnection();
    modbus_status := CheckModbusConnection();
    profinet_status := CheckProfinetConnection();

    IF opcua_status <> last_opcua_status THEN
        last_opcua_status := opcua_status;
        IF NOT opcua_status THEN
            ALARM := TRUE;
            LogAuditTrail('OPC UA', 'Connection lost', 1);
        END_IF;
    END_IF;

    IF modbus_status <> last_modbus_status THEN
        last_modbus_status := modbus_status;
        IF NOT modbus_status THEN
            ALARM := TRUE;
            LogAuditTrail('Modbus', 'Connection lost', 2);
        END_IF;
    END_IF;

    IF profinet_status <> last_profinet_status THEN
        last_profinet_status := profinet_status;
        IF NOT profinet_status THEN
            ALARM := TRUE;
            LogAuditTrail('Profinet', 'Connection lost', 3);
        END_IF;
    END_IF;

    RETURN TRUE;
END_METHOD

END_FUNCTION_BLOCK



