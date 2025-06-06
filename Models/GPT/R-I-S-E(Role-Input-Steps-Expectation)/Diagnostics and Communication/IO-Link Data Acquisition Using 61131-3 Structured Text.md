FUNCTION_BLOCK FB_IOL_Read5Values
VAR_INPUT
    StartRead      : BOOL;     // Trigger to start read operation
    IO_LinkAddr    : ARRAY[1..5] OF INT; // Logical channel IDs or address indexes
END_VAR

VAR_OUTPUT
    Value          : ARRAY[1..5] OF REAL;   // Read values from IO-Link master
    Status         : ARRAY[1..5] OF INT;    // 0=OK, 1=Timeout, 2=CRC Error, 3=Comm Fault
    Busy           : BOOL;                  // TRUE when reading is in progress
    Done           : BOOL;                  // TRUE when all reads are completed
    ErrorGlobal    : BOOL;                  // TRUE if any read has failed
END_VAR

VAR
    i              : INT := 1;              // Loop index
    ReadStep       : INT := 0;              // State machine step
    RetryCounter   : ARRAY[1..5] OF INT;    // Retry counter for each value
    MaxRetries     : INT := 2;              // Max number of retries
    InternalTrigger: BOOL;                 // Internal trigger to control read sequence
END_VAR
