FUNCTION_BLOCK IO_LINK_DATA_ACQUISITION
VAR_INPUT
    IO_Link_Master_Interface : REFERENCE TO IO_LINK_MASTER_INTERFACE; // Reference to the IO-Link Master Interface
END_VAR

VAR_OUTPUT
    ProcessValues : ARRAY[1..5] OF REAL; // Array to store the process values
    ReadStatus : ARRAY[1..5] OF INT;     // Status code for each read operation (0 = Success, -1 = Timeout, -2 = Device Fault)
END_VAR

VAR
    LastProcessValues : ARRAY[1..5] OF REAL;
    LastReadStatus : ARRAY[1..5] OF INT;
    RetryCount : ARRAY[1..5] OF INT; // Retry count for each value
    MaxRetries : INT := 3;           // Maximum number of retries
    Timer : TON;                     // Timer for timeout handling
END_VAR

// Method to read a single process value from the IO-Link master
METHOD ReadProcessValue : BOOL
VAR_INPUT
    ValueIndex : INT;
END_VAR
VAR
    TempValue : REAL;
    Result : INT;
END_VAR
    // Simulate reading a process value from the IO-Link master
    // In a real system, this would involve actual communication with the master
    CASE ValueIndex OF
        1:
            TempValue := 100.0 + Random(10); // Simulated value
            Result := 0; // Success
        2:
            TempValue := 200.0 + Random(10); // Simulated value
            Result := 0; // Success
        3:
            TempValue := 300.0 + Random(10); // Simulated value
            Result := 0; // Success
        4:
            TempValue := 400.0 + Random(10); // Simulated value
            Result := -1; // Timeout simulation
        5:
            TempValue := 500.0 + Random(10); // Simulated value
            Result := 0; // Success
        ELSE
            Result := -2; // Invalid index
    END_CASE;

    IF Result = 0 THEN
        ProcessValues[ValueIndex] := TempValue;
        ReadStatus[ValueIndex] := 0; // Success
        RetryCount[ValueIndex] := 0; // Reset retry count on success
        RETURN TRUE;
    ELSIF Result = -1 THEN
        IF RetryCount[ValueIndex] < MaxRetries THEN
            RetryCount[ValueIndex] := RetryCount[ValueIndex] + 1;
            RETURN FALSE; // Retry
        ELSE
            ReadStatus[ValueIndex] := -1; // Timeout after max retries
            RetryCount[ValueIndex] := 0; // Reset retry count
            RETURN FALSE; // Fail
        END_IF;
    ELSIF Result = -2 THEN
        ReadStatus[ValueIndex] := -2; // Device fault
        RetryCount[ValueIndex] := 0; // Reset retry count
        RETURN FALSE; // Fail
    END_IF;

    RETURN FALSE; // Default case
END_METHOD

// Main execution logic
FOR i := 1 TO SIZEOF(ProcessValues, 1) DO
    IF NOT ReadProcessValue(i) THEN
        // Handle failure: log or take other actions
        PRINTF('Failed to read ProcessValue %d, Status: %d\n', i, ReadStatus[i]);
    END_IF;
END_FOR;



