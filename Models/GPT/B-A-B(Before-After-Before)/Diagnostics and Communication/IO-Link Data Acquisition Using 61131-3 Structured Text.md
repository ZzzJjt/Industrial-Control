FUNCTION_BLOCK IOLINK_PROCESS_READER
VAR_INPUT
    ENABLE        : BOOL;            // Start read operation
    CHANNEL_ID    : ARRAY[1..5] OF INT;  // IO-Link channel addresses
END_VAR

VAR_OUTPUT
    VALUE         : ARRAY[1..5] OF REAL;  // Process values
    STATUS        : ARRAY[1..5] OF INT;   // 0 = OK, 1 = Timeout, 2 = Comm Error
    DONE          : BOOL;                 // TRUE when all reads are complete
    ERROR         : BOOL;                 // TRUE if any read fails
END_VAR

VAR
    ChannelIndex  : INT := 1;
    ReadInProgress: BOOL := FALSE;
    RetryCount    : INT := 0;
    MAX_RETRIES   : INT := 3;
    ReadTimer     : TON;

    // Simulated IOLink read block (vendor-specific in real deployment)
    FUNCTION_BLOCK IO_LINK_READ
    VAR_INPUT
        CHANNEL : INT;
        TRIGGER : BOOL;
    END_VAR
    VAR_OUTPUT
        READY   : BOOL;
        VALUE   : REAL;
        STATUS  : INT;   // 0 = OK, 1 = Timeout, 2 = CommError
    END_VAR
END_VAR

VAR
    IOLinkFB : IO_LINK_READ;
    Triggered : BOOL := FALSE;
END_VAR

// Main logic
IF ENABLE THEN
    DONE := FALSE;
    ERROR := FALSE;

    // Trigger read for current channel
    IF NOT ReadInProgress THEN
        IOLinkFB.CHANNEL := CHANNEL_ID[ChannelIndex];
        IOLinkFB.TRIGGER := TRUE;
        ReadTimer(IN := TRUE, PT := T#500ms); // Timeout guard
        ReadInProgress := TRUE;
        RetryCount := 0;
    ELSE
        IOLinkFB.TRIGGER := FALSE; // Prevent repeated edge
    END_IF

    // If result is ready
    IF IOLinkFB.READY THEN
        VALUE[ChannelIndex] := IOLinkFB.VALUE;
        STATUS[ChannelIndex] := IOLinkFB.STATUS;
        ReadInProgress := FALSE;
        ReadTimer(IN := FALSE);
        ChannelIndex := ChannelIndex + 1;
    ELSIF ReadTimer.Q THEN
        // Retry if timeout
        RetryCount := RetryCount + 1;
        IF RetryCount <= MAX_RETRIES THEN
            IOLinkFB.TRIGGER := TRUE;
            ReadTimer(IN := TRUE, PT := T#500ms);
        ELSE
            STATUS[ChannelIndex] := 1; // Timeout
            VALUE[ChannelIndex] := -9999.0;
            ReadInProgress := FALSE;
            ReadTimer(IN := FALSE);
            ChannelIndex := ChannelIndex + 1;
        END_IF
    END_IF

    // All channels done
    IF ChannelIndex > 5 THEN
        DONE := TRUE;
        ChannelIndex := 1;

        // Check if any STATUS[] has failure
        ERROR := FALSE;
        FOR i := 1 TO 5 DO
            IF STATUS[i] <> 0 THEN
                ERROR := TRUE;
            END_IF
        END_FOR
    END_IF

ELSE
    // Reset state machine
    ChannelIndex := 1;
    DONE := FALSE;
    ERROR := FALSE;
    ReadInProgress := FALSE;
    ReadTimer(IN := FALSE);
END_IF
