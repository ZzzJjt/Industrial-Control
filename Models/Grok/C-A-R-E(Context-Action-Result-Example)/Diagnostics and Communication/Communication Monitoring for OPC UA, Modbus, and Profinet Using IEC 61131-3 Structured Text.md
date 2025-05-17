(* IEC 61131-3 Structured Text: COMM_MONITOR Function Block *)
(* Purpose: Monitors OPC UA, Modbus, and Profinet connection status, triggers alarms, and logs failures *)

FUNCTION_BLOCK COMM_MONITOR
VAR_INPUT
    ENABLE : BOOL;                  (* TRUE to enable monitoring *)
    OPC_UA_STATUS : INT;            (* 0: Connected, 1: Timeout, 2: Auth Failure, 3: Server Unreachable *)
    MODBUS_STATUS : INT;            (* 0: Connected, 1: Timeout, 2: CRC Error, 3: Device Unresponsive *)
    PROFINET_STATUS : INT;          (* 0: Connected, 1: Network Failure, 2: Device Offline, 3: Config Error *)
END_VAR
VAR_OUTPUT
    ALARM_OPC_UA : BOOL;            (* TRUE if OPC UA connection fails *)
    ALARM_MODBUS : BOOL;            (* TRUE if Modbus connection fails *)
    ALARM_PROFINET : BOOL;          (* TRUE if Profinet connection fails *)
    AUDIT_LOG : ARRAY[1..100] OF STRING[80]; (* Audit trail entries *)
    LOG_COUNT : INT;                (* Number of log entries *)
    ERROR : BOOL;                   (* TRUE if internal error occurs *)
    ERROR_CODE : INT;               (* 0: No error, 1: Log buffer full, 2: Invalid status *)
END_VAR
VAR
    State : INT := 0;               (* State: 0=Idle, 1=Check, 2=Log *)
    LastStatus : ARRAY[1..3] OF INT; (* Previous status for change detection *)
    Timestamp : STRING[20];         (* Simulated timestamp *)
    LogBufferFull : BOOL;           (* TRUE if log buffer is full *)
END_VAR

(* Main Logic *)
IF NOT ENABLE THEN
    (* Reset outputs when disabled *)
    ALARM_OPC_UA := FALSE;
    ALARM_MODBUS := FALSE;
    ALARM_PROFINET := FALSE;
    ERROR := FALSE;
    ERROR_CODE := 0;
    State := 0;
ELSE
    CASE State OF
        0: (* Idle *)
            State := 1; (* Move to Check *)
        
        1: (* Check Status *)
            (* Validate status inputs *)
            IF OPC_UA_STATUS > 3 OR MODBUS_STATUS > 3 OR PROFINET_STATUS > 3 THEN
                ERROR := TRUE;
                ERROR_CODE := 2; (* Invalid status *)
                State := 0;
            ELSE
                ERROR := FALSE;
                ERROR_CODE := 0;
                
                (* Monitor OPC UA *)
                ALARM_OPC_UA := OPC_UA_STATUS <> 0;
                IF OPC_UA_STATUS <> LastStatus[1] AND OPC_UA_STATUS <> 0 THEN
                    State := 2; (* Log failure *)
                END_IF;
                LastStatus[1] := OPC_UA_STATUS;
                
                (* Monitor Modbus *)
                ALARM_MODBUS := MODBUS_STATUS <> 0;
                IF MODBUS_STATUS <> LastStatus[2] AND MODBUS_STATUS <> 0 THEN
                    State := 2; (* Log failure *)
                END_IF;
                LastStatus[2] := MODBUS_STATUS;
                
                (* Monitor Profinet *)
                ALARM_PROFINET := PROFINET_STATUS <> 0;
                IF PROFINET_STATUS <> LastStatus[3] AND PROFINET_STATUS <> 0 THEN
                    State := 2; (* Log failure *)
                END_IF;
                LastStatus[3] := PROFINET_STATUS;
                
                IF State <> 2 THEN
                    State := 1; (* Continue checking *)
                END_IF;
            END_IF;
        
        2: (* Log Failure *)
            IF LOG_COUNT >= 100 THEN
                LogBufferFull := TRUE;
                ERROR := TRUE;
                ERROR_CODE := 1; (* Log buffer full *)
                State := 1; (* Return to Check *)
            ELSE
                LogBufferFull := FALSE;
                LOG_COUNT := LOG_COUNT + 1;
                
                (* Simulate timestamp: 2025-05-17 18:06:00 *)
                Timestamp := '2025-05-17 18:06:00'; (* Replace with system clock *)
                
                (* Log OPC UA failure *)
                IF OPC_UA_STATUS <> LastStatus[1] AND OPC_UA_STATUS <> 0 THEN
                    CASE OPC_UA_STATUS OF
                        1: AUDIT_LOG[LOG_COUNT] := CONCAT(Timestamp, ' OPC UA Timeout – Error Code 0x01');
                        2: AUDIT_LOG[LOG_COUNT] := CONCAT(Timestamp, ' OPC UA Auth Failure – Error Code 0x02');
                        3: AUDIT_LOG[LOG_COUNT] := CONCAT(Timestamp, ' OPC UA Server Unreachable – Error Code 0x03');
                    END_CASE;
                END_IF;
                
                (* Log Modbus failure *)
                IF MODBUS_STATUS <> LastStatus[2] AND MODBUS_STATUS <> 0 THEN
                    CASE MODBUS_STATUS OF
                        1: AUDIT_LOG[LOG_COUNT] := CONCAT(Timestamp, ' Modbus Timeout – Error Code 0x01');
                        2: AUDIT_LOG[LOG_COUNT] := CONCAT(Timestamp, ' Modbus CRC Error – Error Code 0x02');
                        3: AUDIT_LOG[LOG_COUNT] := CONCAT(Timestamp, ' Modbus Device Unresponsive – Error Code 0x03');
                    END_CASE;
                END_IF;
                
                (* Log Profinet failure *)
                IF PROFINET_STATUS <> LastStatus[3] AND PROFINET_STATUS <> 0 THEN
                    CASE PROFINET_STATUS OF
                        1: AUDIT_LOG[LOG_COUNT] := CONCAT(Timestamp, ' Profinet Network Failure – Error Code 0x01');
                        2: AUDIT_LOG[LOG_COUNT] := CONCAT(Timestamp, ' Profinet Device Offline – Error Code 0x02');
                        3: AUDIT_LOG[LOG_COUNT] := CONCAT(Timestamp, ' Profinet Config Error – Error Code 0x03');
                    END_CASE;
                END_IF;
                
                State := 1; (* Return to Check *)
            END_IF;
    END_CASE;
END_IF;

(* Notes:
   - Purpose: Monitors OPC UA, Modbus, and Profinet connections, triggers alarms, and logs failures.
   - Inputs:
     - ENABLE: TRUE to enable monitoring.
     - OPC_UA_STATUS: 0 (Connected), 1 (Timeout), 2 (Auth Failure), 3 (Server Unreachable).
     - MODBUS_STATUS: 0 (Connected), 1 (Timeout), 2 (CRC Error), 3 (Device Unresponsive).
     - PROFINET_STATUS: 0 (Connected), 1 (Network Failure), 2 (Device Offline), 3 (Config Error).
   - Outputs:
     - ALARM_OPC_UA, ALARM_MODBUS, ALARM_PROFINET: TRUE on failure.
     - AUDIT_LOG: ARRAY[1..100] OF STRING[80], stores timestamped failure logs.
     - LOG_COUNT: Number of log entries.
     - ERROR: TRUE if internal error occurs.
     - ERROR_CODE: 0 (No error), 1 (Log buffer full), 2 (Invalid status).
   - Logic:
     - State 1 (Check): Validates status, sets alarms, detects changes.
     - State 2 (Log): Adds failure to AUDIT_LOG with timestamp, protocol, reason, error code.
     - Concurrently monitors all protocols each scan cycle.
   - Error Handling:
     - Invalid status: >3 for any protocol (ERROR_CODE=2).
     - Log buffer full: LOG_COUNT >= 100 (ERROR_CODE=1).
   - Safety:
     - Bounded log array (100 entries) prevents overflow.
     - State machine ensures scan-cycle safety.
     - Input validation avoids invalid status processing.
   - Traceability:
     - Timestamped logs (e.g., "2025-05-17 18:06:00 OPC UA Timeout – Error Code 0x01").
     - AUDIT_LOG supports HMI/SCADA export.
   - Usage:
     - OPC UA timeout: Sets ALARM_OPC_UA, logs "OPC UA Timeout – Error Code 0x01".
     - Continues monitoring Modbus/Profinet without interruption.
   - Maintenance:
     - Modular design aids debugging.
     - ERROR_CODE and AUDIT_LOG support diagnostics.
   - Platform Notes:
     - Assumes STRING[80] for logs; adjust for platform limits.
     - Timestamp is static ("2025-05-17 18:06:00"); replace with system clock (e.g., GET_SYSTEM_TIME).
     - Log buffer size (100) is practical; adjustable for memory constraints.
*)
END_FUNCTION_BLOCK
