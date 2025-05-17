(* IEC 61131-3 Structured Text: POWERLINK_DIAG Function Block *)
(* Purpose: Retrieves and logs diagnostic information from Ethernet PowerLink Control Nodes *)

FUNCTION_BLOCK POWERLINK_DIAG
VAR_INPUT
    ENABLE : BOOL;                  (* TRUE to enable cyclic diagnostics *)
    NODE_ID : USINT;                (* Control Node ID, 1–240 *)
    EXECUTE : BOOL;                 (* Trigger manual diagnostic retrieval on rising edge *)
END_VAR
VAR_OUTPUT
    COMM_STATUS : INT;              (* 0: OK, 1: Timeout, 2: CRC Error, 3: Node Unresponsive *)
    ERROR_CODE : INT;               (* 0: No error, 1: Node Offline, 2: Invalid Data, 3: Network Failure *)
    NODE_HEALTH : INT;              (* 0: Healthy, 1: Warning, 2: Critical *)
    DIAG_LOG : ARRAY[1..50] OF STRING[80]; (* Diagnostic log entries *)
    LOG_COUNT : INT;                (* Number of log entries *)
    BUSY : BOOL;                    (* TRUE while processing *)
    ERROR : BOOL;                   (* TRUE if internal error occurs *)
    ERROR_CODE_FB : INT;            (* 0: No error, 1: Invalid Node ID, 2: Log buffer full, 3: Driver error *)
END_VAR
VAR
    State : INT := 0;               (* State: 0=Idle, 1=Request, 2=Process, 3=Log *)
    LastExecute : BOOL;             (* Previous EXECUTE state for edge detection *)
    CyclicTimer : TON;              (* 1-second cyclic timer *)
    LastStatus : INT;               (* Previous COMM_STATUS for change detection *)
    Timestamp : STRING[20];         (* Simulated timestamp *)
    LogBufferFull : BOOL;           (* TRUE if log buffer is full *)
END_VAR

(* Main Logic *)
IF NOT ENABLE AND NOT EXECUTE THEN
    (* Reset outputs when disabled *)
    COMM_STATUS := 0;
    ERROR_CODE := 0;
    NODE_HEALTH := 0;
    BUSY := FALSE;
    ERROR := FALSE;
    ERROR_CODE_FB := 0;
    State := 0;
    CyclicTimer.IN := FALSE;
ELSE
    (* Cyclic execution *)
    IF ENABLE AND NOT EXECUTE THEN
        CyclicTimer(IN := TRUE, PT := T#1s);
        IF CyclicTimer.Q THEN
            CyclicTimer.IN := FALSE; (* Reset timer *)
            State := 1; (* Start diagnostic request *)
        END_IF;
    END_IF;
    
    (* Manual execution on rising edge *)
    IF EXECUTE AND NOT LastExecute THEN
        State := 1; (* Start diagnostic request *)
    END_IF;
    LastExecute := EXECUTE;
    
    CASE State OF
        0: (* Idle *)
            BUSY := FALSE;
            IF ENABLE OR EXECUTE THEN
                State := 1; (* Move to Request *)
            END_IF;
        
        1: (* Request Diagnostics *)
            BUSY := TRUE;
            (* Validate NODE_ID *)
            IF NODE_ID < 1 OR NODE_ID > 240 THEN
                ERROR := TRUE;
                ERROR_CODE_FB := 1; (* Invalid Node ID *)
                BUSY := FALSE;
                State := 0;
            ELSE
                (* Simulate PowerLink driver data retrieval *)
                (* Replace with actual driver call, e.g., EPL_GET_DIAG(NODE_ID) *)
                IF RAND() MOD 100 < 5 THEN (* 5% chance of driver error *)
                    ERROR := TRUE;
                    ERROR_CODE_FB := 3; (* Driver error *)
                    COMM_STATUS := 0;
                    ERROR_CODE := 0;
                    NODE_HEALTH := 0;
                    BUSY := FALSE;
                    State := 0;
                ELSE
                    (* Simulated diagnostic data *)
                    COMM_STATUS := RAND() MOD 4; (* 0: OK, 1: Timeout, 2: CRC Error, 3: Unresponsive *)
                    IF COMM_STATUS = 0 THEN
                        ERROR_CODE := 0;
                        NODE_HEALTH := 0; (* Healthy *)
                    ELSIF COMM_STATUS = 1 THEN
                        ERROR_CODE := 1; (* Node Offline *)
                        NODE_HEALTH := 1; (* Warning *)
                    ELSIF COMM_STATUS = 2 THEN
                        ERROR_CODE := 2; (* Invalid Data *)
                        NODE_HEALTH := 1; (* Warning *)
                    ELSE
                        ERROR_CODE := 3; (* Network Failure *)
                        NODE_HEALTH := 2; (* Critical *)
                    END_IF;
                    State := 2; (* Move to Process *)
                END_IF;
            END_IF;
        
        2: (* Process Diagnostics *)
            IF COMM_STATUS <> LastStatus AND COMM_STATUS <> 0 THEN
                State := 3; (* Log failure *)
            ELSE
                BUSY := FALSE;
                State := 0; (* Return to Idle *)
            END_IF;
            LastStatus := COMM_STATUS;
        
        3: (* Log Failure *)
            IF LOG_COUNT >= 50 THEN
                LogBufferFull := TRUE;
                ERROR := TRUE;
                ERROR_CODE_FB := 2; (* Log buffer full *)
                BUSY := FALSE;
                State := 0;
            ELSE
                LogBufferFull := FALSE;
                LOG_COUNT := LOG_COUNT + 1;
                
                (* Simulate timestamp: 2025-05-17 18:06:00 *)
                Timestamp := '2025-05-17 18:06:00'; (* Replace with system clock *)
                
                (* Log entry *)
                CASE COMM_STATUS OF
                    1: DIAG_LOG[LOG_COUNT] := CONCAT(Timestamp, CONCAT(' Node ', 
                              CONCAT(TO_STRING(NODE_ID), ' Timeout – Error 0x01, Health: Warning')));
                    2: DIAG_LOG[LOG_COUNT] := CONCAT(Timestamp, CONCAT(' Node ', 
                              CONCAT(TO_STRING(NODE_ID), ' CRC Error – Error 0x02, Health: Warning')));
                    3: DIAG_LOG[LOG_COUNT] := CONCAT(Timestamp, CONCAT(' Node ', 
                              CONCAT(TO_STRING(NODE_ID), ' Unresponsive – Error 0x03, Health: Critical')));
                END_CASE;
                
                BUSY := FALSE;
                State := 0; (* Return to Idle *)
            END_IF;
    END_CASE;
END_IF;

(* Notes:
   - Purpose: Monitors Ethernet PowerLink Control Node diagnostics via Managing Node.
   - Inputs:
     - ENABLE: TRUE for cyclic diagnostics (every 1s).
     - NODE_ID: Control Node ID (1–240).
     - EXECUTE: Triggers manual diagnostic retrieval on rising edge.
   - Outputs:
     - COMM_STATUS: 0 (OK), 1 (Timeout), 2 (CRC Error), 3 (Node Unresponsive).
     - ERROR_CODE: 0 (No error), 1 (Node Offline), 2 (Invalid Data), 3 (Network Failure).
     - NODE_HEALTH: 0 (Healthy), 1 (Warning), 2 (Critical).
     - DIAG_LOG: ARRAY[1..50] OF STRING[80], stores timestamped diagnostic logs.
     - LOG_COUNT: Number of log entries.
     - BUSY: TRUE while processing.
     - ERROR: TRUE if internal error occurs.
     - ERROR_CODE_FB: 0 (No error), 1 (Invalid Node ID), 2 (Log buffer full), 3 (Driver error).
   - Logic:
     - State 1 (Request): Validates NODE_ID, retrieves diagnostics (simulated).
     - State 2 (Process): Interprets status, sets health and error codes.
     - State 3 (Log): Adds failure to DIAG_LOG with timestamp, node ID, status, error, health.
     - Cyclic operation every 1s when ENABLE=TRUE; manual trigger via EXECUTE.
   - Error Handling:
     - Invalid Node ID: <1 or >240 (ERROR_CODE_FB=1).
     - Log buffer full: LOG_COUNT >= 50 (ERROR_CODE_FB=2).
     - Driver error: Simulated 5% failure (ERROR_CODE_FB=3).
   - Safety:
     - Bounded log array (50 entries) prevents overflow.
     - State machine ensures scan-cycle safety (e.g., 10–100 ms cycles).
     - Input validation avoids invalid NODE_ID processing.
   - Traceability:
     - Timestamped logs (e.g., "2025-05-17 18:06:00 Node 10 Timeout – Error 0x01, Health: Warning").
     - DIAG_LOG supports HMI/SCADA export.
   - Usage:
     - Node timeout: Sets COMM_STATUS=1, NODE_HEALTH=1, logs "Node 10 Timeout – Error 0x01".
     - Continues monitoring other nodes via NODE_ID cycling.
   - Maintenance:
     - Modular design aids debugging.
     - ERROR_CODE_FB and DIAG_LOG support diagnostics.
   - Platform Notes:
     - Assumes STRING[80] for logs, USINT for NODE_ID; adjust for platform limits.
     - Timestamp is static ("2025-05-17 18:06:00"); replace with system clock (e.g., GET_SYSTEM_TIME).
     - Diagnostic data is simulated; replace RAND() with PowerLink driver call (e.g., EPL_GET_DIAG).
     - Log buffer size (50) is practical; adjustable for memory constraints.
*)
END_FUNCTION_BLOCK
