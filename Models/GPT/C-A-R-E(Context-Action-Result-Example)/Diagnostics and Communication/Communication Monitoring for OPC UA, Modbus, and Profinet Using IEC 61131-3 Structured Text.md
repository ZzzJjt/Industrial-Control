FUNCTION_BLOCK FB_ProtocolMonitor
VAR_INPUT
    OPC_UA_Status     : BOOL;   // TRUE = connected, FALSE = failed
    Modbus_Status     : BOOL;
    Profinet_Status   : BOOL;
    CurrentTime       : DT;     // Timestamp for logging
END_VAR

VAR_OUTPUT
    Alarm_OPC_UA      : BOOL;   // Alarm for OPC UA failure
    Alarm_Modbus      : BOOL;
    Alarm_Profinet    : BOOL;
END_VAR

VAR
    Prev_OPC_UA       : BOOL := TRUE;
    Prev_Modbus       : BOOL := TRUE;
    Prev_Profinet     : BOOL := TRUE;
END_VAR

// Log structure (this would typically map to a logging system or file)
TYPE LogEntry :
STRUCT
    Timestamp    : DT;
    Protocol     : STRING[16];
    Description  : STRING[64];
    ErrorCode    : WORD;
END_STRUCT
END_TYPE

VAR
    LogBuffer : ARRAY[1..100] OF LogEntry;
    LogIndex  : INT := 1;
END_VAR

// --- OPC UA Monitoring ---
IF (NOT OPC_UA_Status) AND (Prev_OPC_UA) THEN
    Alarm_OPC_UA := TRUE;
    
    // Log the error
    IF LogIndex <= 100 THEN
        LogBuffer[LogIndex].Timestamp := CurrentTime;
        LogBuffer[LogIndex].Protocol := 'OPC UA';
        LogBuffer[LogIndex].Description := 'Timeout or server unreachable';
        LogBuffer[LogIndex].ErrorCode := 16#05; // Example code
        LogIndex := LogIndex + 1;
    END_IF
ELSIF OPC_UA_Status THEN
    Alarm_OPC_UA := FALSE;
END_IF
Prev_OPC_UA := OPC_UA_Status;

// --- Modbus Monitoring ---
IF (NOT Modbus_Status) AND (Prev_Modbus) THEN
    Alarm_Modbus := TRUE;
    IF LogIndex <= 100 THEN
        LogBuffer[LogIndex].Timestamp := CurrentTime;
        LogBuffer[LogIndex].Protocol := 'Modbus';
        LogBuffer[LogIndex].Description := 'No response from slave';
        LogBuffer[LogIndex].ErrorCode := 16#01; // Example code
        LogIndex := LogIndex + 1;
    END_IF
ELSIF Modbus_Status THEN
    Alarm_Modbus := FALSE;
END_IF
Prev_Modbus := Modbus_Status;

// --- Profinet Monitoring ---
IF (NOT Profinet_Status) AND (Prev_Profinet) THEN
    Alarm_Profinet := TRUE;
    IF LogIndex <= 100 THEN
        LogBuffer[LogIndex].Timestamp := CurrentTime;
        LogBuffer[LogIndex].Protocol := 'Profinet';
        LogBuffer[LogIndex].Description := 'IO device lost or response timeout';
        LogBuffer[LogIndex].ErrorCode := 16#09; // Example code
        LogIndex := LogIndex + 1;
    END_IF
ELSIF Profinet_Status THEN
    Alarm_Profinet := FALSE;
END_IF
Prev_Profinet := Profinet_Status;
