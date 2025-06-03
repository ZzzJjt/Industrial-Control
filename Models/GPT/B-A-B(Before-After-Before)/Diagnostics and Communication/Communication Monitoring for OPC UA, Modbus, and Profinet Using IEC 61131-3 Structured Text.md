FUNCTION_BLOCK COMM_MONITOR
VAR_INPUT
    OPCUA_Status     : INT;   // 0 = OK, otherwise = error code
    Modbus_Status    : INT;
    Profinet_Status  : INT;
    ENABLE           : BOOL;
END_VAR

VAR_OUTPUT
    OPCUA_Alarm      : BOOL;
    Modbus_Alarm     : BOOL;
    Profinet_Alarm   : BOOL;
    OPCUA_ErrorCode  : INT;
    Modbus_ErrorCode : INT;
    Profinet_ErrorCode : INT;
    NewAuditEntry    : BOOL;      // TRUE when new audit entry is available
    AuditText        : STRING[255]; // Descriptive log for maintenance system
END_VAR

VAR
    LastOPCUA_Status    : INT := 0;
    LastModbus_Status   : INT := 0;
    LastProfinet_Status : INT := 0;

    ErrorMsg : STRING[100];
    TimeNow  : DT;
    Protocol : STRING[10];
    Changed  : BOOL;
END_VAR

// Reset outputs
OPCUA_Alarm := FALSE;
Modbus_Alarm := FALSE;
Profinet_Alarm := FALSE;
NewAuditEntry := FALSE;

// Check only when enabled
IF ENABLE THEN
    // OPC UA
    IF OPCUA_Status <> 0 AND OPCUA_Status <> LastOPCUA_Status THEN
        OPCUA_Alarm := TRUE;
        OPCUA_ErrorCode := OPCUA_Status;
        Protocol := 'OPCU-UA';
        ErrorMsg := CONCAT('Connection failure. Code: ', INT_TO_STRING(OPCUA_Status));
        Changed := TRUE;
    END_IF
    LastOPCUA_Status := OPCUA_Status;

    // Modbus
    IF Modbus_Status <> 0 AND Modbus_Status <> LastModbus_Status THEN
        Modbus_Alarm := TRUE;
        Modbus_ErrorCode := Modbus_Status;
        Protocol := 'Modbus';
        ErrorMsg := CONCAT('Connection failure. Code: ', INT_TO_STRING(Modbus_Status));
        Changed := TRUE;
    END_IF
    LastModbus_Status := Modbus_Status;

    // Profinet
    IF Profinet_Status <> 0 AND Profinet_Status <> LastProfinet_Status THEN
        Profinet_Alarm := TRUE;
        Profinet_ErrorCode := Profinet_Status;
        Protocol := 'Profinet';
        ErrorMsg := CONCAT('Connection failure. Code: ', INT_TO_STRING(Profinet_Status));
        Changed := TRUE;
    END_IF
    LastProfinet_Status := Profinet_Status;

    // Create audit log entry
    IF Changed THEN
        TimeNow := DT#2025-01-01-00:00:00; // Replace with actual RTC or system time source
        AuditText := CONCAT(Protocol, CONCAT(' Error at ', DT_TO_STRING(TimeNow)));
        AuditText := CONCAT(AuditText, CONCAT('. Detail: ', ErrorMsg));
        NewAuditEntry := TRUE;
        Changed := FALSE;
    END_IF
END_IF
