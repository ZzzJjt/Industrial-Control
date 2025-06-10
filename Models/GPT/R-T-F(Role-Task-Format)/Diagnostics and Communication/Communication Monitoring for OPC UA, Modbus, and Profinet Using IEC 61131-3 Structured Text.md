FUNCTION_BLOCK COMM_MONITOR
VAR_INPUT
    OPC_Connected     : BOOL;     // TRUE = OPC UA connected
    Modbus_Connected  : BOOL;     // TRUE = Modbus connected
    Profinet_Connected: BOOL;     // TRUE = Profinet connected

    OPC_ErrorCode     : INT;      // Error code for OPC UA
    Modbus_ErrorCode  : INT;      // Error code for Modbus
    Profinet_ErrorCode: INT;      // Error code for Profinet

    SystemTime        : TIME;     // Simulated system time input
    Enable            : BOOL;     // Trigger to activate monitoring
END_VAR

VAR_OUTPUT
    Alarm_OPC_UA      : BOOL;
    Alarm_Modbus      : BOOL;
    Alarm_Profinet    : BOOL;

    AuditLog_OPC_UA      : STRING(255);
    AuditLog_Modbus      : STRING(255);
    AuditLog_Profinet    : STRING(255);
END_VAR

VAR
    Prev_OPC_Connected     : BOOL := TRUE;
    Prev_Modbus_Connected  : BOOL := TRUE;
    Prev_Profinet_Connected: BOOL := TRUE;

    TempString : STRING(100);
END_VAR

// Monitoring logic begins
IF Enable THEN

    // OPC UA Monitoring
    IF NOT OPC_Connected AND Prev_OPC_Connected THEN
        Alarm_OPC_UA := TRUE;
        TempString := CONCAT('OPC UA FAIL, Code: ', INT_TO_STRING(OPC_ErrorCode));
        AuditLog_OPC_UA := CONCAT(CONCAT(CONCAT('[' , TIME_TO_STRING(SystemTime)), '] '), TempString);
    ELSIF OPC_Connected AND NOT Prev_OPC_Connected THEN
        Alarm_OPC_UA := FALSE;
        AuditLog_OPC_UA := CONCAT(CONCAT(CONCAT('[' , TIME_TO_STRING(SystemTime)), '] OPC UA RECOVERED'), '');
    END_IF
    Prev_OPC_Connected := OPC_Connected;

    // Modbus Monitoring
    IF NOT Modbus_Connected AND Prev_Modbus_Connected THEN
        Alarm_Modbus := TRUE;
        TempString := CONCAT('Modbus FAIL, Code: ', INT_TO_STRING(Modbus_ErrorCode));
        AuditLog_Modbus := CONCAT(CONCAT(CONCAT('[' , TIME_TO_STRING(SystemTime)), '] '), TempString);
    ELSIF Modbus_Connected AND NOT Prev_Modbus_Connected THEN
        Alarm_Modbus := FALSE;
        AuditLog_Modbus := CONCAT(CONCAT(CONCAT('[' , TIME_TO_STRING(SystemTime)), '] Modbus RECOVERED'), '');
    END_IF
    Prev_Modbus_Connected := Modbus_Connected;

    // Profinet Monitoring
    IF NOT Profinet_Connected AND Prev_Profinet_Connected THEN
        Alarm_Profinet := TRUE;
        TempString := CONCAT('Profinet FAIL, Code: ', INT_TO_STRING(Profinet_ErrorCode));
        AuditLog_Profinet := CONCAT(CONCAT(CONCAT('[' , TIME_TO_STRING(SystemTime)), '] '), TempString);
    ELSIF Profinet_Connected AND NOT Prev_Profinet_Connected THEN
        Alarm_Profinet := FALSE;
        AuditLog_Profinet := CONCAT(CONCAT(CONCAT('[' , TIME_TO_STRING(SystemTime)), '] Profinet RECOVERED'), '');
    END_IF
    Prev_Profinet_Connected := Profinet_Connected;

END_IF
