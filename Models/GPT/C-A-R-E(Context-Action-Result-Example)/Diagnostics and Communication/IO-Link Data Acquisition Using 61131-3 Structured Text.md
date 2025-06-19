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


// Rising-edge detection
IF Enable AND NOT EdgeDetector THEN
    ReadInProgress := TRUE;
    Index := 1;
    RetryCounter := 0;
    CommDone := FALSE;
    CommError := FALSE;
END_IF
EdgeDetector := Enable;

IF ReadInProgress THEN
    // Attempt to read value
    InternalSuccess := IOLink_Read(
        MasterID := MasterAddress,
        Channel  := Index,
        Value    := InternalValue
    );

    IF InternalSuccess THEN
        Values[Index] := InternalValue;
        Status[Index] := TRUE;
        Index := Index + 1;
        RetryCounter := 0;
    ELSE
        RetryCounter := RetryCounter + 1;
        IF RetryCounter >= MaxRetries THEN
            Values[Index] := -9999; // Error placeholder
            Status[Index] := FALSE;
            CommError := TRUE;
            Index := Index + 1;
            RetryCounter := 0;
        END_IF
    END_IF

    // End condition
    IF Index > 5 THEN
        ReadInProgress := FALSE;
        CommDone := TRUE;
    END_IF
END_IF

