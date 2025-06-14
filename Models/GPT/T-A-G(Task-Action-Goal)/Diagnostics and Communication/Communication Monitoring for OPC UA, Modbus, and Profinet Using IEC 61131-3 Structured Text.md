FUNCTION_BLOCK FB_CommMonitor
VAR_INPUT
    OPC_UA_Status    : BOOL;
    Modbus_Status    : BOOL;
    Profinet_Status  : BOOL;
END_VAR

VAR_OUTPUT
    Alarm_OPC_UA     : BOOL;
    Alarm_Modbus     : BOOL;
    Alarm_Profinet   : BOOL;
    Audit_Trail      : STRING[255];
    Error_OPC_UA     : INT;
    Error_Modbus     : INT;
    Error_Profinet   : INT;
END_VAR

VAR
    prev_OPC_UA      : BOOL := TRUE;
    prev_Modbus      : BOOL := TRUE;
    prev_Profinet    : BOOL := TRUE;
    timeStamp        : STRING[20];
END_VAR

(* Initialize outputs *)
Alarm_OPC_UA := FALSE;
Alarm_Modbus := FALSE;
Alarm_Profinet := FALSE;
Audit_Trail := '';
Error_OPC_UA := 0;
Error_Modbus := 0;
Error_Profinet := 0;

(* Check OPC UA *)
IF NOT OPC_UA_Status THEN
    Alarm_OPC_UA := TRUE;
    IF prev_OPC_UA THEN
        Error_OPC_UA := 1001; // Custom error code for OPC UA failure
        timeStamp := GetSystemTimestamp();
        Audit_Trail := CONCAT(timeStamp, ': OPC UA failure - Error 1001');
    END_IF
END_IF
prev_OPC_UA := OPC_UA_Status;

(* Check Modbus *)
IF NOT Modbus_Status THEN
    Alarm_Modbus := TRUE;
    IF prev_Modbus THEN
        Error_Modbus := 2001; // Custom error code for Modbus failure
        timeStamp := GetSystemTimestamp();
        Audit_Trail := CONCAT(timeStamp, ': Modbus failure - Error 2001');
    END_IF
END_IF
prev_Modbus := Modbus_Status;

(* Check Profinet *)
IF NOT Profinet_Status THEN
    Alarm_Profinet := TRUE;
    IF prev_Profinet THEN
        Error_Profinet := 3001; // Custom error code for Profinet failure
        timeStamp := GetSystemTimestamp();
        Audit_Trail := CONCAT(timeStamp, ': Profinet failure - Error 3001');
    END_IF
END_IF
prev_Profinet := Profinet_Status;

(* Safety check: limit audit string size *)
IF LEN(Audit_Trail) > 250 THEN
    Audit_Trail := LEFT(Audit_Trail, 250);
END_IF
