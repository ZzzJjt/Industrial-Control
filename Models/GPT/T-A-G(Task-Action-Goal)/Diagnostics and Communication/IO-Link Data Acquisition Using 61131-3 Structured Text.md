FUNCTION_BLOCK FB_IOlink_Reader
VAR_INPUT
    TriggerRead     : BOOL;
    MasterAddress   : BYTE;
END_VAR

VAR_OUTPUT
    Value           : ARRAY[1..5] OF INT;
    StatusOK        : ARRAY[1..5] OF BOOL;
    CommError       : BOOL;
    ErrorCode       : INT;
END_VAR

VAR
    prevTrigger     : BOOL := FALSE;
    i               : INT;
    readSuccess     : BOOL;
    tempVal         : INT;
    retries         : INT;
    maxRetries      : INT := 3;
END_VAR

// Default outputs
CommError := FALSE;
ErrorCode := 0;
FOR i := 1 TO 5 DO
    StatusOK[i] := FALSE;
END_FOR

// Trigger on rising edge of TriggerRead
IF TriggerRead AND NOT prevTrigger THEN
    FOR i := 1 TO 5 DO
        retries := 0;
        readSuccess := FALSE;

        WHILE (retries < maxRetries) AND NOT readSuccess DO
            tempVal := IOlinkRead(MasterAddress, i);  // Placeholder function
            readSuccess := ValidateIOData(tempVal);
            retries := retries + 1;
        END_WHILE

        IF readSuccess THEN
            Value[i] := tempVal;
            StatusOK[i] := TRUE;
        ELSE
            CommError := TRUE;
            ErrorCode := 100 + i; // Unique code per channel failure
        END_IF
    END_FOR
END_IF

prevTrigger := TriggerRead;

// Dummy data validator
FUNCTION ValidateIOData : BOOL
VAR_INPUT
    val : INT;
END_VAR
ValidateIOData := (val >= 0) AND (val <= 32767);

// Simulated IO-Link read function
FUNCTION IOlinkRead : INT
VAR_INPUT
    addr : BYTE;
    channel : INT;
END_VAR
// Simulate good data return
IOlinkRead := 1000 + channel * 100;
