FUNCTION_BLOCK FB_ReadIOLinkValues
VAR_INPUT
    Enable          : BOOL;         // Rising-edge trigger to start reading
    MasterAddress   : INT;          // Address or ID of the IO-Link master
END_VAR
VAR_OUTPUT
    Values          : ARRAY [1..5] OF INT;    // Retrieved process values
    Status          : ARRAY [1..5] OF BOOL;   // TRUE = OK, FALSE = Error
    CommDone        : BOOL;                   // Indicates read cycle complete
    CommError       : BOOL;                   // TRUE if any value failed
END_VAR
VAR
    ReadInProgress  : BOOL;
    Index           : INT;
    RetryCounter    : INT;
    MaxRetries      : INT := 3;
    InternalValue   : INT;
    InternalSuccess : BOOL;
    EdgeDetector    : BOOL;
END_VAR
