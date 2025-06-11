TYPE E_ReadStatus :
(
    SUCCESS = 0,
    TIMEOUT = 1,
    CRC_ERROR = 2,
    INVALID_CHANNEL = 3,
    COMM_FAILURE = 4,
    RETRYING = 5
);
END_TYPE

FUNCTION_BLOCK FB_IO_LinkReader
VAR_INPUT
    ENABLE: BOOL := FALSE;                      // Enable cyclic reads
    TRIGGER_READ: BOOL := FALSE;                // Manual trigger to start reading
    CHANNELS: ARRAY [0..4] OF UINT := [1,2,3,4,5]; // IO-Link channel numbers
    MAX_RETRIES: UINT := 3;                     // Max retry attempts on failure
    RETRY_INTERVAL: TIME := T#500ms;            // Delay between retries
END_VAR

VAR_OUTPUT
    VALUES: ARRAY [0..4] OF INT;                // Process values read from each channel
    STATUSES: ARRAY [0..4] OF E_ReadStatus;     // Read statuses per channel
    ALL_SUCCESS: BOOL := FALSE;                 // TRUE if all reads succeeded
    ANY_FAILURE: BOOL := FALSE;                 // TRUE if any read failed
    ERROR_COUNT: UINT := 0;                     // Total number of failed reads
    LAST_ERROR: E_ReadStatus := SUCCESS;        // Last recorded error
END_VAR

VAR
    state: UINT := 0;
    index: INT := 0;
    retryCount: UINT := 0;
    retryTimer: TON;
    readActive: BOOL := FALSE;
    tempValue: INT := 0;
    tempStatus: E_ReadStatus := SUCCESS;
END_VAR

// Reset outputs at start of scan
ALL_SUCCESS := TRUE;
ANY_FAILURE := FALSE;
ERROR_COUNT := 0;

IF NOT ENABLE THEN
    RETURN;
END_IF;

// Start reading on trigger or after retry delay
IF TRIGGER_READ OR (retryTimer.Q AND retryCount > 0) THEN
    retryTimer(IN := FALSE); // Reset timer
    readActive := TRUE;
    index := 0;
    state := 0;
    TRIGGER_READ := FALSE;
END_IF;

CASE state OF
    0: // Begin reading channels
        IF index <= 4 THEN
            retryCount := 0;
            tempStatus := ReadIO_LinkChannel(CHANNELS[index], ADR(tempValue));
            VALUES[index] := tempValue;
            STATUSES[index] := tempStatus;

            CASE tempStatus OF
                SUCCESS:
                    ; // Do nothing
                TIMEOUT, CRC_ERROR, INVALID_CHANNEL, COMM_FAILURE:
                    ANY_FAILURE := TRUE;
                    ALL_SUCCESS := FALSE;
                    ERROR_COUNT := ERROR_COUNT + 1;
                    LAST_ERROR := tempStatus;
                    retryCount := retryCount + 1;
                    state := 1; // Enter retry loop
                    EXIT; // Exit case to retry logic
            END_CASE;

            index := index + 1;
        ELSE
            // All channels processed
            state := 99;
        END_IF;

    1: // Retry failed channel
        retryTimer(IN := TRUE, PT := RETRY_INTERVAL);
        IF retryTimer.Q THEN
            retryTimer(IN := FALSE);
            tempStatus := ReadIO_LinkChannel(CHANNELS[index], ADR(tempValue));
            VALUES[index] := tempValue;
            STATUSES[index] := tempStatus;

            CASE tempStatus OF
                SUCCESS:
                    ; // Success on retry
                TIMEOUT, CRC_ERROR, INVALID_CHANNEL, COMM_FAILURE:
                    retryCount := retryCount + 1;
                    LAST_ERROR := tempStatus;
            END_CASE;

            IF tempStatus = SUCCESS OR retryCount >= MAX_RETRIES THEN
                IF tempStatus <> SUCCESS THEN
                    ANY_FAILURE := TRUE;
                    ALL_SUCCESS := FALSE;
                    ERROR_COUNT := ERROR_COUNT + 1;
                END_IF;
                index := index + 1;
                state := 0; // Resume next channel
            END_IF;
        END_IF;

    99: // Finalize and reset
        readActive := FALSE;
        TRIGGER_READ := FALSE;
        retryCount := 0;
END_CASE;

FUNCTION GetReadStatusDescription : STRING(20)
VAR_INPUT
    status: E_ReadStatus;
END_VAR

CASE status OF
    SUCCESS:         GetReadStatusDescription := 'Success';
    TIMEOUT:         GetReadStatusDescription := 'Timeout';
    CRC_ERROR:       GetReadStatusDescription := 'CRC Error';
    INVALID_CHANNEL: GetReadStatusDescription := 'Invalid Channel';
    COMM_FAILURE:    GetReadStatusDescription := 'Comm Failure';
    RETRYING:        GetReadStatusDescription := 'Retrying...';
END_CASE;

PROGRAM PLC_PRG
VAR
    ioLinkReader: FB_IO_LinkReader;
    channelIDs: ARRAY [0..4] OF UINT := [1,2,3,4,5];
    diagValues: ARRAY [0..4] OF INT;
    diagStatuses: ARRAY [0..4] OF E_ReadStatus;
END_VAR

ioLinkReader(
    ENABLE := TRUE,
    TRIGGER_READ := manualTriggerInput, // Optional manual trigger
    CHANNELS := channelIDs,
    MAX_RETRIES := 3
);

// Copy results for further use
diagValues := ioLinkReader.VALUES;
diagStatuses := ioLinkReader.STATUSES;

// HMI/Alarm Integration
IF ioLinkReader.ANY_FAILURE THEN
    hmiErrorText := CONCAT('IO-Link Read Error: ', GetReadStatusDescription(ioLinkReader.LAST_ERROR));
    hmiAlarmActive := TRUE;
ELSE
    hmiAlarmActive := FALSE;
END_IF;
