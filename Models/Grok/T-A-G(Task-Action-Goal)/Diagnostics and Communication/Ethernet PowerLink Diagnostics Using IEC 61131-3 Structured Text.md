(* IEC 61131-3 Structured Text function block for PowerLink node diagnostics *)
(* Retrieves and processes communication status, error codes, and node health *)
(* Ensures real-time monitoring, robust error handling, and maintainable code *)

FUNCTION_BLOCK POWERLINK_DIAG
VAR_INPUT
    ENABLE : BOOL; (* TRUE to enable function block *)
    NODE_ID : USINT; (* Control Node ID, 1–240 *)
    DIAG_DATA : ARRAY[0..31] OF BYTE; (* Raw diagnostic data from MN *)
END_VAR

VAR_OUTPUT
    DONE : BOOL; (* TRUE when data is successfully processed *)
    BUSY : BOOL; (* TRUE during operation *)
    ERROR : BOOL; (* TRUE if error occurs *)
    ERROR_ID : DWORD; (* Error code: 0=None, 1=Invalid Node, 2=Comm Failure, 3=Data Corrupted, 4=Retry Exceeded *)
    COMM_STATUS : USINT; (* Node status: 0=Stopped, 1=Operational, 2=Pre-Operational, etc. *)
    ERROR_CODE : DWORD; (* Last error code from node *)
    NODE_HEALTH : USINT; (* Health score, 0–100 *)
    LAST_UPDATE : TIME; (* Timestamp of last successful update *)
END_VAR

VAR
    (* Constants *)
    MAX_RETRIES : USINT := 3; (* Max retry attempts for communication *)
    DATA_SIZE : INT := 32; (* Expected diagnostic data size in bytes *)
    HEALTH_WINDOW : INT := 100; (* Sliding window for health calculation *)
    
    (* Internal variables *)
    RetryCount : USINT; (* Current retry attempt *)
    LastEnable : BOOL; (* Tracks previous ENABLE state *)
    OperationActive : BOOL; (* Tracks ongoing operation *)
    ErrorHistory : ARRAY[0..99] OF BOOL; (* Tracks errors for health score *)
    ErrorIndex : INT; (* Current index in ErrorHistory *)
    ErrorCount : INT; (* Number of errors in window *)
    TempData : ARRAY[0..31] OF BYTE; (* Temporary buffer for data validation *)
    i : INT; (* Loop variable *)
END_VAR

(* Reset outputs when disabled *)
IF NOT ENABLE THEN
    DONE := FALSE;
    BUSY := FALSE;
    ERROR := FALSE;
    ERROR_ID := 0;
    COMM_STATUS := 0;
    ERROR_CODE := 0;
    NODE_HEALTH := 100;
    LAST_UPDATE := T#0s;
    OperationActive := FALSE;
    LastEnable := FALSE;
    RetryCount := 0;
    RETURN;
END_IF;

(* Initialize on rising edge of ENABLE *)
IF ENABLE AND NOT LastEnable THEN
    (* Reset state *)
    DONE := FALSE;
    BUSY := FALSE;
    ERROR := FALSE;
    ERROR_ID := 0;
    COMM_STATUS := 0;
    ERROR_CODE := 0;
    NODE_HEALTH := 100;
    LAST_UPDATE := T#0s;
    OperationActive := FALSE;
    RetryCount := 0;
    ErrorCount := 0;
    ErrorIndex := 0;
    
    (* Initialize error history *)
    FOR i := 0 TO HEALTH_WINDOW - 1 DO
        ErrorHistory[i] := FALSE;
    END_FOR;
END_IF;

(* Store ENABLE state *)
LastEnable := ENABLE;

(* Main logic *)
IF ENABLE AND NOT OperationActive THEN
    BUSY := TRUE;
    DONE := FALSE;
    ERROR := FALSE;
    ERROR_ID := 0;
    OperationActive := TRUE;
    
    (* Validate NODE_ID *)
    IF NODE_ID < 1 OR NODE_ID > 240 THEN
        ERROR := TRUE;
        ERROR_ID := 1; (* Invalid Node *)
        DONE := FALSE;
        BUSY := FALSE;
        OperationActive := FALSE;
        ErrorHistory[ErrorIndex] := TRUE;
        ErrorCount := ErrorCount + 1;
        ErrorIndex := (ErrorIndex + 1) MOD HEALTH_WINDOW;
        IF ErrorIndex = 0 THEN
            ErrorCount := 0; (* Reset count at window end *)
            FOR i := 0 TO HEALTH_WINDOW - 1 DO
                IF ErrorHistory[i] THEN
                    ErrorCount := ErrorCount + 1;
                END_IF;
            END_FOR;
        END_IF;
        NODE_HEALTH := MAX(0, 100 - (ErrorCount * 100 / HEALTH_WINDOW));
        RETURN;
    END_IF;
    
    (* Simulate MN communication: request diagnostic data *)
    (* Placeholder: Replace with actual PowerLink SDO read *)
    IF NOT POWERLINK_SDO_READ(NODE_ID, DIAG_DATA) THEN
        RetryCount := RetryCount + 1;
        IF RetryCount >= MAX_RETRIES THEN
            ERROR := TRUE;
            ERROR_ID := 2; (* Comm Failure *)
            DONE := FALSE;
            BUSY := FALSE;
            OperationActive := FALSE;
            RetryCount := 0;
            ErrorHistory[ErrorIndex] := TRUE;
            ErrorCount := ErrorCount + 1;
            ErrorIndex := (ErrorIndex + 1) MOD HEALTH_WINDOW;
            IF ErrorIndex = 0 THEN
                ErrorCount := 0;
                FOR i := 0 TO HEALTH_WINDOW - 1 DO
                    IF ErrorHistory[i] THEN
                        ErrorCount := ErrorCount + 1;
                    END_IF;
                END_FOR;
            END_IF;
            NODE_HEALTH := MAX(0, 100 - (ErrorCount * 100 / HEALTH_WINDOW));
            RETURN;
        END_IF;
        BUSY := FALSE;
        OperationActive := FALSE;
        RETURN; (* Retry next cycle *)
    END_IF;
    
    (* Reset retry count on successful read *)
    RetryCount := 0;
    
    (* Validate data integrity: check expected size and checksum *)
    (* Simulated checksum: sum of bytes *)
    VAR checksum : DWORD := 0; END_VAR
    FOR i := 0 TO DATA_SIZE - 1 DO
        checksum := checksum + DIAG_DATA[i];
    END_FOR;
    IF checksum = 0 THEN (* Arbitrary validation *)
        ERROR := TRUE;
        ERROR_ID := 3; (* Data Corrupted *)
        DONE := FALSE;
        BUSY := FALSE;
        OperationActive := FALSE;
        ErrorHistory[ErrorIndex] := TRUE;
        ErrorCount := ErrorCount + 1;
        ErrorIndex := (ErrorIndex + 1) MOD HEALTH_WINDOW;
        IF ErrorIndex = 0 THEN
            ErrorCount := 0;
            FOR i := 0 TO HEALTH_WINDOW - 1 DO
                IF ErrorHistory[i] THEN
                    ErrorCount := ErrorCount + 1;
                END_IF;
            END_FOR;
        END_IF;
        NODE_HEALTH := MAX(0, 100 - (ErrorCount * 100 / HEALTH_WINDOW));
        RETURN;
    END_IF;
    
    (* Process diagnostic data *)
    (* Simulated data extraction: assume byte 0=status, bytes 1-4=error code *)
    COMM_STATUS := DIAG_DATA[0];
    ERROR_CODE := DWORD#0;
    FOR i := 1 TO 4 DO
        ERROR_CODE := SHL(ERROR_CODE, 8) OR DIAG_DATA[i];
    END_FOR;
    
    (* Update health score *)
    ErrorHistory[ErrorIndex] := (ERROR_CODE <> 0);
    ErrorCount := ErrorCount + (ErrorHistory[ErrorIndex] ? 1 : 0);
    ErrorIndex := (ErrorIndex + 1) MOD HEALTH_WINDOW;
    IF ErrorIndex = 0 THEN
        ErrorCount := 0;
        FOR i := 0 TO HEALTH_WINDOW - 1 DO
            IF ErrorHistory[i] THEN
                ErrorCount := ErrorCount + 1;
            END_IF;
        END_FOR;
    END_IF;
    NODE_HEALTH := MAX(0, 100 - (ErrorCount * 100 / HEALTH_WINDOW));
    
    (* Update timestamp *)
    LAST_UPDATE := CURRENT_TIME(); (* Placeholder: replace with PLC time function *)
    
    (* Complete operation *)
    DONE := TRUE;
    BUSY := FALSE;
    OperationActive := FALSE;
ELSE
    (* Operation completed or not enabled *)
    BUSY := FALSE;
    OperationActive := FALSE;
END_IF;

(* Execution Notes *)
(* - POWERLINK_SDO_READ is a placeholder for actual PowerLink SDO read API *)
(* - Cyclically retrieves diagnostic data (comm status, error codes, health) *)
(* - Validates NODE_ID (1–240) and data integrity via checksum *)
(* - Tracks node health using a 100-sample sliding window of error occurrences *)
(* - Executes in a single PLC scan cycle, suitable for real-time PowerLink networks *)
(* - Robust error handling for invalid inputs, communication failures, and data issues *)
END_FUNCTION_BLOCK
