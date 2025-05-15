FUNCTION_BLOCK COMMUNICATION_MONITOR
VAR_INPUT
    OPC_UA_Status : INT;  // Status code for OPC UA connection (0 = Connected, non-zero = Error Code)
    MODBUS_Status : INT;   // Status code for Modbus connection (0 = Connected, non-zero = Error Code)
    PROFIBUS_Status : INT; // Status code for Profibus connection (0 = Connected, non-zero = Error Code)
END_VAR

VAR_OUTPUT
    AlarmActive : BOOL;    // TRUE if any protocol has an active alarm
    AuditLog : STRING[255];// Latest audit log entry
END_VAR

VAR
    LastOPCUAStatus : INT := -1;
    LastMODBUSStatus : INT := -1;
    LastPROFIBUSStatus : INT := -1;
    CurrentTime : TIME_OF_DAY;
END_VAR

METHOD LogFailure : BOOL
VAR_INPUT
    Protocol : STRING[20];
    ErrorCode : INT;
END_VAR
VAR
    Timestamp : STRING[20];
END_VAR
    // Get current time
    CurrentTime := TP_TO_TOD(T_PLC_TIME);
    Timestamp := CONCAT(
        TO_STRING(CurrentTime.hour), ':',
        TO_STRING(CurrentTime.min), ':',
        TO_STRING(CurrentTime.sec), '.',
        TO_STRING(CurrentTime.msec)
    );

    // Create audit log entry
    AuditLog := CONCAT(
        Timestamp, ' - ',
        Protocol, ' timeout - Error Code ', TO_STRING(ErrorCode)
    );

    RETURN TRUE;
END_METHOD

// Main execution logic
AlarmActive := FALSE;

// Check OPC UA status
IF OPC_UA_Status <> 0 AND OPC_UA_Status <> LastOPCUAStatus THEN
    AlarmActive := LogFailure('OPC UA', OPC_UA_Status);
    LastOPCUAStatus := OPC_UA_Status;
ELSIF OPC_UA_Status = 0 THEN
    LastOPCUAStatus := OPC_UA_Status;
END_IF;

// Check Modbus status
IF MODBUS_Status <> 0 AND MODBUS_Status <> LastMODBUSStatus THEN
    AlarmActive := LogFailure('Modbus', MODBUS_Status);
    LastMODBUSStatus := MODBUS_Status;
ELSIF MODBUS_Status = 0 THEN
    LastMODBUSStatus := MODBUS_Status;
END_IF;

// Check Profibus status
IF PROFIBUS_Status <> 0 AND PROFIBUS_Status <> LastPROFIBUSStatus THEN
    AlarmActive := LogFailure('Profibus', PROFIBUS_Status);
    LastPROFIBUSStatus := PROFIBUS_Status;
ELSIF PROFIBUS_Status = 0 THEN
    LastPROFIBUSStatus := PROFIBUS_Status;
END_IF;



