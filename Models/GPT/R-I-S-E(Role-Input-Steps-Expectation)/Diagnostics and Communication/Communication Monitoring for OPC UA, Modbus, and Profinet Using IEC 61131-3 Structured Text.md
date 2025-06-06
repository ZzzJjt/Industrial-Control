FUNCTION_BLOCK FB_CommMonitor
VAR_INPUT
    OPCUA_Connected      : BOOL;    // TRUE if OPC UA is connected
    OPCUA_ErrorCode      : INT;     // OPC UA error code (0 = OK)

    Modbus_Connected     : BOOL;    // TRUE if Modbus is connected
    Modbus_ErrorCode     : INT;     // Modbus error code (0 = OK)

    Profinet_Connected   : BOOL;    // TRUE if Profinet is connected
    Profinet_ErrorCode   : INT;     // Profinet error code (0 = OK)

    CurrentTime          : TIME;    // Input timestamp for logging
END_VAR

VAR_OUTPUT
    OPCUA_Alarm          : BOOL;
    Modbus_Alarm         : BOOL;
    Profinet_Alarm       : BOOL;

    AuditLog_Trigger     : BOOL;    // Trigger to log current AuditLog_Entry
    AuditLog_Entry       : STRING;  // Latest formatted audit entry (reset after use)
END_VAR

VAR
    OPCUA_PrevStatus     : BOOL := TRUE;
    Modbus_PrevStatus    : BOOL := TRUE;
    Profinet_PrevStatus  : BOOL := TRUE;

    TempStr              : STRING;
END_VAR


// === OPC UA Monitoring ===
IF (NOT OPCUA_Connected) AND (OPCUA_PrevStatus) THEN
    OPCUA_Alarm := TRUE;
    AuditLog_Trigger := TRUE;
    AuditLog_Entry := CONCAT(CONCAT('[OPCUA][', TIME_TO_STRING(CurrentTime)), 
        CONCAT('] Connection LOST. Code: ', INT_TO_STRING(OPCUA_ErrorCode)));
ELSIF OPCUA_Connected AND (NOT OPCUA_PrevStatus) THEN
    OPCUA_Alarm := FALSE;
    AuditLog_Trigger := TRUE;
    AuditLog_Entry := CONCAT(CONCAT('[OPCUA][', TIME_TO_STRING(CurrentTime)), '] Connection RESTORED.');
END_IF;
OPCUA_PrevStatus := OPCUA_Connected;


// === Modbus Monitoring ===
IF (NOT Modbus_Connected) AND (Modbus_PrevStatus) THEN
    Modbus_Alarm := TRUE;
    AuditLog_Trigger := TRUE;
    AuditLog_Entry := CONCAT(CONCAT('[Modbus][', TIME_TO_STRING(CurrentTime)), 
        CONCAT('] Connection LOST. Code: ', INT_TO_STRING(Modbus_ErrorCode)));
ELSIF Modbus_Connected AND (NOT Modbus_PrevStatus) THEN
    Modbus_Alarm := FALSE;
    AuditLog_Trigger := TRUE;
    AuditLog_Entry := CONCAT(CONCAT('[Modbus][', TIME_TO_STRING(CurrentTime)), '] Connection RESTORED.');
END_IF;
Modbus_PrevStatus := Modbus_Connected;


// === Profinet Monitoring ===
IF (NOT Profinet_Connected) AND (Profinet_PrevStatus) THEN
    Profinet_Alarm := TRUE;
    AuditLog_Trigger := TRUE;
    AuditLog_Entry := CONCAT(CONCAT('[Profinet][', TIME_TO_STRING(CurrentTime)), 
        CONCAT('] Connection LOST. Code: ', INT_TO_STRING(Profinet_ErrorCode)));
ELSIF Profinet_Connected AND (NOT Profinet_PrevStatus) THEN
    Profinet_Alarm := FALSE;
    AuditLog_Trigger := TRUE;
    AuditLog_Entry := CONCAT(CONCAT('[Profinet][', TIME_TO_STRING(CurrentTime)), '] Connection RESTORED.');
END_IF;
Profinet_PrevStatus := Profinet_Connected;
