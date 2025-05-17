(* IEC 61131-3 Structured Text: IOLINK_READ_VALUES Function Block *)
(* Purpose: Reads five process values from an IO-Link master with error handling and logging *)

FUNCTION_BLOCK IOLINK_READ_VALUES
VAR_INPUT
    ENABLE : BOOL;                  (* TRUE to enable cyclic reading *)
    EXECUTE : BOOL;                 (* Trigger manual read on rising edge *)
    MASTER_ID : USINT;              (* IO-Link master ID, 1–255 *)
    RETRY_COUNT : USINT;            (* Number of retry attempts per value *)
END_VAR
VAR_OUTPUT
    VALUES : ARRAY[1..5] OF REAL;   (* Process values *)
    STATUS : ARRAY[1..5] OF INT;    (* 0: OK, 1: Timeout, 2: Invalid Data, 3: Device Fault *)
    BUSY : BOOL;                    (* TRUE while reading *)
    DONE : BOOL;                    (* TRUE when read cycle completes *)
    ERROR : BOOL;                   (* TRUE if error occurs *)
    ERROR_CODE : INT;               (* 0: No error, 1: Invalid Master ID, 2: Comm Failure, 3: Retry Exhausted *)
    DIAG_LOG : ARRAY[1..50] OF STRING[80]; (* Error log entries *)
    LOG_COUNT : INT;                (* Number of log entries *)
END_VAR
VAR
    State : INT := 0;               (* State: 0=Idle, 1=Request, 2=Read, 3=Log *)
    LastExecute : BOOL;             (* Previous EXECUTE state for edge detection *)
    CyclicTimer : TON;              (* 100ms cyclic timer *)
    CurrentValueIdx : INT;          (* Current value being read, 1–5 *)
    RetryCounter : INT;             (* Retry attempts for current value *)
    LastStatus : ARRAY[1..5] OF INT; (* Previous STATUS for change detection *)
    Timestamp : STRING[20];         (* Simulated timestamp *)
    LogBufferFull : BOOL;           (* TRUE if log buffer is full *)
END_VAR

(* Main Logic *)
IF NOT ENABLE AND NOT EXECUTE THEN
    (* Reset outputs when disabled *)
    FOR i := 1 TO 5 DO
        VALUES[i] := 0.0;
        STATUS[i] := 0;
        LastStatus[i] := 0;
    END_FOR;
    BUSY := FALSE;
    DONE := FALSE;
    ERROR := FALSE;
    ERROR_CODE := 0;
    State := 0;
    CyclicTimer.IN := FALSE;
    CurrentValueIdx := 0;
    RetryCounter := 0;
ELSE
    (* Cyclic execution *)
    IF ENABLE AND NOT EXECUTE THEN
        CyclicTimer(IN := TRUE, PT := T#100ms);
        IF CyclicTimer.Q THEN
            CyclicTimer.IN := FALSE; (* Reset timer *)
            State := 1; (* Start read cycle *)
            CurrentValueIdx := 1;
            RetryCounter := 0;
        END_IF;
    END_IF;
    
    (* Manual execution on rising edge *)
    IF EXECUTE AND NOT LastExecute THEN
        State := 1; (* Start read cycle *)
        CurrentValueIdx := 1;
        RetryCounter := 0;
    END_IF;
    LastExecute := EXECUTE;
    
    CASE State OF
        0: (* Idle *)
            BUSY := FALSE;
            IF ENABLE OR EXECUTE THEN
                State := 1; (* Move to Request *)
                CurrentValueIdx := 1;
                RetryCounter := 0;
            END_IF;
        
        1: (* Request Read *)
            BUSY := TRUE;
            (* Validate MASTER_ID *)
            IF MASTER_ID < 1 OR MASTER_ID > 255 THEN
                ERROR := TRUE;
                ERROR_CODE := 1; (* Invalid Master ID *)
                BUSY := FALSE;
                DONE := TRUE;
                State := 0;
            ELSE
                (* Simulate IO-Link master communication *)
                (* Replace with actual driver call, e.g., IOL_READ_VALUE(MASTER_ID, CurrentValueIdx) *)
                IF RAND() MOD 100 < 5 THEN (* 5% chance of comm failure *)
                    ERROR := TRUE;
                    ERROR_CODE := 2; (* Comm Failure *)
                    STATUS[CurrentValueIdx] := 1; (* Timeout *)
                    State := 2; (* Process *)
                ELSE
                    (* Simulated data *)
                    VALUES[CurrentValueIdx] := 100.0 + RAND() MOD 10; (* e.g., 100–109 *)
                    STATUS[CurrentValueIdx] := RAND() MOD 4; (* 0: OK, 1: Timeout, 2: Invalid Data, 3: Fault *)
                    IF STATUS[CurrentValueIdx] <> 0 THEN
                        RetryCounter := RetryCounter + 1;
                        IF RetryCounter >= RETRY_COUNT THEN
                            ERROR := TRUE;
                            ERROR_CODE := 3; (* Retry Exhausted *)
                            State := 2; (* Process *)
                        ELSE
                            State := 1; (* Retry *)
                        END_IF;
                    ELSE
                        RetryCounter := 0;
                        State := 2; (* Process *)
                    END_IF;
                END_IF;
            END_IF;
        
        2: (* Process Read *)
            IF STATUS[CurrentValueIdx] <> LastStatus[CurrentValueIdx] AND STATUS[CurrentValueIdx] <> 0 THEN
                State := 3; (* Log failure *)
            ELSE
                LastStatus[CurrentValueIdx] := STATUS[CurrentValueIdx];
                IF CurrentValueIdx < 5 THEN
                    CurrentValueIdx := CurrentValueIdx + 1;
                    RetryCounter := 0;
                    State := 1; (* Read next value *)
                ELSE
                    BUSY := FALSE;
                    DONE := TRUE;
                    State := 0; (* Complete cycle *)
                END_IF;
            END_IF;
        
        3: (* Log Failure *)
            IF LOG_COUNT >= 50 THEN
                LogBufferFull := TRUE;
                ERROR := TRUE;
                ERROR_CODE := 2; (* Log buffer full *)
                BUSY := FALSE;
                State := 0;
            ELSE
                LogBufferFull := FALSE;
                LOG_COUNT := LOG_COUNT + 1;
                
                (* Simulate timestamp: 2025-05-17 18:06:00 *)
                Timestamp := '2025-05-17 18:06:00'; (* Replace with system clock *)
                
                (* Log entry *)
                CASE STATUS[CurrentValueIdx] OF
                    1: DIAG_LOG[LOG_COUNT] := CONCAT(Timestamp, CONCAT(' Master ', 
                              CONCAT(TO_STRING(MASTER_ID), CONCAT(' Value ', 
                              CONCAT(TO_STRING(CurrentValueIdx), ' Timeout – Error 0x01')))));
                    2: DIAG_LOG[LOG_COUNT] := CONCAT(Timestamp, CONCAT(' Master ', 
                              CONCAT(TO_STRING(MASTER_ID), CONCAT(' Value ', 
                              CONCAT(TO_STRING(CurrentValueIdx), ' Invalid Data – Error 0x02')))));
                    3: DIAG_LOG[LOG_COUNT] := CONCAT(Timestamp, CONCAT(' Master ', 
                              CONCAT(TO_STRING(MASTER_ID), CONCAT(' Value ', 
                              CONCAT(TO_STRING(CurrentValueIdx), ' Device Fault – Error 0x03')))));
                END_CASE;
                
                LastStatus[CurrentValueIdx] := STATUS[CurrentValueIdx];
                IF CurrentValueIdx < 5 THEN
                    CurrentValueIdx := CurrentValueIdx + 1;
                    RetryCounter := 0;
                    State := 1; (* Read next value *)
                ELSE
                    BUSY := FALSE;
                    DONE := TRUE;
                    State := 0; (* Complete cycle *)
                END_IF;
            END_IF;
    END_CASE;
END_IF;

(* Notes:
   - Purpose: Reads five process values from an IO-Link master with error handling and logging.
   - Inputs:
     - ENABLE: TRUE for cyclic reading (every 100ms).
     - EXECUTE: Triggers manual read on rising edge.
     - MASTER_ID: IO-Link master ID (1–255).
     - RETRY_COUNT: Number of retry attempts per value.
   - Outputs:
     - VALUES: ARRAY[1..5] OF REAL, process values.
     - STATUS: ARRAY[1..5] OF INT, 0 (OK), 1 (Timeout), 2 (Invalid Data), 3 (Device Fault).
     - BUSY: TRUE while reading.
     - DONE: TRUE when read cycle completes.
     - ERROR: TRUE if error occurs.
     - ERROR_CODE: 0 (No error), 1 (Invalid Master ID), 2 (Comm Failure), 3 (Retry Exhausted).
     - DIAG_LOG: ARRAY[1..50] OF STRING[80], error logs.
     - LOG_COUNT: Number of log entries.
   - Logic:
     - State 1 (Request): Validates MASTER_ID, reads value (simulated).
     - State 2 (Process): Sets STATUS, retries on failure up to RETRY_COUNT.
     - State 3 (Log): Logs failures with timestamp, master ID, value index, error.
     - Cyclic operation every 100ms when ENABLE=TRUE; manual via EXECUTE.
   - Error Handling:
     - Invalid Master ID: <1 or >255 (ERROR_CODE=1).
     - Comm Failure: Simulated 5% failure (ERROR_CODE=2).
     - Retry Exhausted: After RETRY_COUNT attempts (ERROR_CODE=3).
     - Log buffer full: LOG_COUNT >= 50 (ERROR_CODE=2).
   - Safety:
     - Bounded log array (50 entries) prevents overflow.
     - State machine ensures scan-cycle safety (e.g., 10–100 ms cycles).
     - Input validation avoids invalid MASTER_ID processing.
   - Traceability:
     - Timestamped logs (e.g., "2025-05-17 18:06:00 Master 1 Value 3 Timeout – Error 0x01").
     - DIAG_LOG supports HMI/SCADA export.
   - Usage:
     - Timeout on value 3: Sets STATUS[3]=1, retries, logs "Master 1 Value 3 Timeout – Error 0x01".
     - Delivers valid values 1, 2, 4, 5 for controller action (e.g., retry, alarm).
   - Maintenance:
     - Modular design aids debugging.
     - ERROR_CODE and DIAG_LOG support diagnostics.
   - Platform Notes:
     - Assumes STRING[80] for logs, USINT for MASTER_ID; adjust for platform limits.
     - Timestamp is static ("2025-05-17 18:06:00"); replace with system clock (e.g., GET_SYSTEM_TIME).
     - Data is simulated; replace RAND() with IO-Link driver call (e.g., IOL_READ_VALUE).
     - Log buffer size (50) is practical; adjustable for memory constraints.
*)
END_FUNCTION_BLOCK
