(* Function Block: IOLINK_READ_PROCESS_DATA
   Purpose: Reads five process values from a remote IO-Link master, with status reporting and error handling.
   Features:
   - Initiates read requests for specified IO-Link channels
   - Stores process values in output array
   - Provides status flags for each read (Success, Timeout, Comm Error)
   - Includes retry logic for failed reads
   - Supports up to 3 retry attempts per channel
*)

FUNCTION_BLOCK IOLINK_READ_PROCESS_DATA
VAR_INPUT
    EXECUTE : BOOL;               (* Rising edge triggers read operation *)
    MASTER_ID : UINT;             (* IO-Link master identifier *)
    CHANNELS : ARRAY[0..4] OF UINT; (* Channel numbers for 5 process values *)
    RETRY_COUNT : UINT := 3;      (* Max retry attempts per channel *)
    TIMEOUT : TIME := T#500ms;    (* Timeout for each read attempt *)
END_VAR

VAR_OUTPUT
    DONE : BOOL;                  (* TRUE when all reads complete successfully *)
    BUSY : BOOL;                  (* TRUE during read operation *)
    ERROR : BOOL;                 (* TRUE if any read fails after retries *)
    ERROR_CODE : UINT;            (* 0=None, 1=Invalid Channel, 2=Comm Error, 3=Timeout *)
    PROCESS_VALUES : ARRAY[0..4] OF REAL; (* Read process values *)
    READ_STATUS : ARRAY[0..4] OF UINT; (* Status: 0=Success, 1=Timeout, 2=Comm Error *)
END_VAR

VAR
    LastExecute : BOOL;           (* Tracks previous EXECUTE state for edge detection *)
    CurrentChannel : UINT;        (* Current channel being processed *)
    RetryAttempts : UINT;         (* Current retry count for channel *)
    ReadTimer : TON;              (* Timer for read timeout *)
    OperationComplete : BOOL;     (* Flag for completed read cycle *)
    ReadResponse : UINT;          (* Response from IO-Link master *)
    TempValue : REAL;             (* Temporary storage for read value *)
END_VAR

(* Initialize outputs *)
DONE := FALSE;
BUSY := FALSE;
ERROR := FALSE;
ERROR_CODE := 0;
FOR i := 0 TO 4 DO
    PROCESS_VALUES[i] := 0.0;
    READ_STATUS[i] := 0;
END_FOR

(* Main logic *)
IF EXECUTE AND NOT LastExecute THEN
    (* Initialize on rising edge *)
    BUSY := TRUE;
    DONE := FALSE;
    ERROR := FALSE;
    ERROR_CODE := 0;
    CurrentChannel := 0;
    RetryAttempts := 0;
    OperationComplete := FALSE;
    ReadTimer(IN := FALSE);
    FOR i := 0 TO 4 DO
        PROCESS_VALUES[i] := 0.0;
        READ_STATUS[i] := 0;
    END_FOR
END_IF

(* Process reads when BUSY *)
IF BUSY AND NOT OperationComplete THEN
    IF CurrentChannel < 5 THEN
        (* Validate channel *)
        IF CHANNELS[CurrentChannel] > 255 THEN (* Assuming max 256 channels *)
            ERROR := TRUE;
            ERROR_CODE := 1; (* Invalid Channel *)
            READ_STATUS[CurrentChannel] := 2; (* Comm Error as fallback *)
            BUSY := FALSE;
            DONE := FALSE;
            RETURN;
        END_IF
        
        (* Start read operation *)
        IF NOT ReadTimer.Q THEN
            ReadTimer(IN := TRUE, PT := TIMEOUT);
            ReadResponse := IOLINK_ReadChannel(MASTER_ID, CHANNELS[CurrentChannel], TempValue);
        END_IF
        
        (* Process response *)
        IF ReadTimer.Q OR ReadResponse <> 255 THEN
            CASE ReadResponse OF
                0: (* Success *)
                    PROCESS_VALUES[CurrentChannel] := TempValue;
                    READ_STATUS[CurrentChannel] := 0; (* Success *)
                    RetryAttempts := 0;
                    CurrentChannel := CurrentChannel + 1;
                    ReadTimer(IN := FALSE);
                1: (* Timeout *)
                    IF RetryAttempts < RETRY_COUNT THEN
                        RetryAttempts := RetryAttempts + 1;
                        ReadTimer(IN := FALSE); (* Reset for retry *)
                    ELSE
                        READ_STATUS[CurrentChannel] := 1; (* Timeout *)
                        ERROR := TRUE;
                        ERROR_CODE := 3; (* Timeout *)
                        CurrentChannel := CurrentChannel + 1;
                        RetryAttempts := 0;
                        ReadTimer(IN := FALSE);
                    END_IF
                2: (* Communication Error *)
                    IF RetryAttempts < RETRY_COUNT THEN
                        RetryAttempts := RetryAttempts + 1;
                        ReadTimer(IN := FALSE); (* Reset for retry *)
                    ELSE
                        READ_STATUS[CurrentChannel] := 2; (* Comm Error *)
                        ERROR := TRUE;
                        ERROR_CODE := 2; (* Comm Error *)
                        CurrentChannel := CurrentChannel + 1;
                        RetryAttempts := 0;
                        ReadTimer(IN := FALSE);
                    END_IF
                ELSE
                    (* Unknown response *)
                    READ_STATUS[CurrentChannel] := 2; (* Comm Error *)
                    ERROR := TRUE;
                    ERROR_CODE := 2; (* Comm Error *)
                    CurrentChannel := CurrentChannel + 1;
                    RetryAttempts := 0;
                    ReadTimer(IN := FALSE);
            END_CASE
        END_IF
    ELSE
        (* All channels processed *)
        OperationComplete := TRUE;
        BUSY := FALSE;
        DONE := NOT ERROR;
    END_IF
END_IF

(* Reset outputs when EXECUTE is FALSE *)
IF NOT EXECUTE THEN
    BUSY := FALSE;
    DONE := FALSE;
    ERROR := FALSE;
    ERROR_CODE := 0;
    CurrentChannel := 0;
    RetryAttempts := 0;
    OperationComplete := FALSE;
    ReadTimer(IN := FALSE);
    FOR i := 0 TO 4 DO
        READ_STATUS[i] := 0;
    END_FOR
END_IF

(* Update edge detection *)
LastExecute := EXECUTE;

(* Simulated IO-Link read function *)
FUNCTION IOLINK_ReadChannel : UINT
VAR_INPUT
    MasterId : UINT;
    Channel : UINT;
END_VAR
VAR_OUTPUT
    Value : REAL;
END_VAR
(* Placeholder: Simulates reading from IO-Link master *)
(* Returns: 0=Success, 1=Timeout, 2=Comm Error, 255=Pending *)
Value := 100.0 + Channel; (* Simulate valid data *)
IOLINK_ReadChannel := 0; (* Simulate success *)
END_FUNCTION
END_FUNCTION_BLOCK
