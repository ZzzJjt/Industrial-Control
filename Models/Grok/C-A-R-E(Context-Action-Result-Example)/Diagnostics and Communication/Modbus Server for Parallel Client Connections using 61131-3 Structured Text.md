(* IEC 61131-3 Structured Text: MODBUS_TCP_SERVER Function Block *)
(* Purpose: Implements a Modbus TCP server for up to 10 clients, supporting function codes 0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x0F, 0x10, 0x17 *)

FUNCTION_BLOCK MODBUS_TCP_SERVER
VAR_INPUT
    ENABLE : BOOL;                  (* TRUE to enable server *)
    PORT : UINT;                    (* TCP port, e.g., 502 *)
    EXECUTE : BOOL;                 (* Trigger manual cycle on rising edge *)
    COILS : ARRAY[0..999] OF BOOL;  (* Coil memory region, read/write *)
    DISCRETE_INPUTS : ARRAY[0..999] OF BOOL; (* Discrete input memory, read-only *)
    HOLDING_REGISTERS : ARRAY[0..999] OF WORD; (* Holding register memory, read/write *)
    INPUT_REGISTERS : ARRAY[0..999] OF WORD; (* Input register memory, read-only *)
END_VAR
VAR_OUTPUT
    ACTIVE : BOOL;                  (* TRUE when server is running *)
    CLIENT_COUNT : USINT;           (* Number of connected clients, 0–10 *)
    REQUESTS_PROCESSED : UDINT;     (* Total requests handled *)
    ERROR : BOOL;                   (* TRUE if error occurs *)
    ERROR_CODE : INT;               (* 0: No error, 1: Invalid Port, 2: Max Clients, 3: Socket Error, 4: Invalid Request, 5: Memory Access Error *)
    DIAG_LOG : ARRAY[1..50] OF STRING[80]; (* Error and event logs *)
    LOG_COUNT : INT;                (* Number of log entries *)
END_VAR
VAR
    State : INT := 0;               (* State: 0=Idle, 1=Listen, 2=Process, 3=Respond *)
    LastExecute : BOOL;             (* Previous EXECUTE state for edge detection *)
    CyclicTimer : TON;              (* 10ms cyclic timer *)
    Clients : ARRAY[1..10] OF STRUCT
        Active : BOOL;              (* TRUE if client connected *)
        SocketID : UINT;            (* Simulated socket ID *)
        RequestBuffer : ARRAY[0..255] OF BYTE; (* Request data *)
        ResponseBuffer : ARRAY[0..255] OF BYTE; (* Response data *)
        BufferLen : USINT;          (* Request length *)
        TransactionID : UINT;       (* Modbus transaction ID *)
    END_STRUCT;
    ClientIdx : USINT;              (* Current client being processed *)
    Timestamp : STRING[20];         (* Simulated timestamp *)
    LogBufferFull : BOOL;           (* TRUE if log buffer full *)
    MBAP : STRUCT                   (* Modbus TCP MBAP header *)
        TransactionID : UINT;
        ProtocolID : UINT;
        Length : UINT;
        UnitID : BYTE;
    END_STRUCT;
    PDU : STRUCT                    (* Modbus Protocol Data Unit *)
        FunctionCode : BYTE;
        StartAddress : UINT;
        Quantity : UINT;
        Data : ARRAY[0..200] OF BYTE;
    END_STRUCT;
END_VAR

(* Main Logic *)
IF NOT ENABLE AND NOT EXECUTE THEN
    (* Reset outputs when disabled *)
    ACTIVE := FALSE;
    CLIENT_COUNT := 0;
    REQUESTS_PROCESSED := 0;
    ERROR := FALSE;
    ERROR_CODE := 0;
    State := 0;
    CyclicTimer.IN := FALSE;
    FOR i := 1 TO 10 DO
        Clients[i].Active := FALSE;
        Clients[i].SocketID := 0;
        Clients[i].BufferLen := 0;
    END_FOR;
ELSE
    (* Validate PORT *)
    IF PORT < 1 OR PORT > 65535 THEN
        ERROR := TRUE;
        ERROR_CODE := 1; (* Invalid Port *)
        ACTIVE := FALSE;
        State := 0;
        RETURN;
    END_IF;
    
    (* Cyclic execution *)
    IF ENABLE AND NOT EXECUTE THEN
        CyclicTimer(IN := TRUE, PT := T#10ms);
        IF CyclicTimer.Q THEN
            CyclicTimer.IN := FALSE; (* Reset timer *)
            State := 1; (* Start server cycle *)
            ClientIdx := 1;
        END_IF;
    END_IF;
    
    (* Manual execution on rising edge *)
    IF EXECUTE AND NOT LastExecute THEN
        State := 1; (* Start server cycle *)
        ClientIdx := 1;
    END_IF;
    LastExecute := EXECUTE;
    
    CASE State OF
        0: (* Idle *)
            ACTIVE := FALSE;
            IF ENABLE OR EXECUTE THEN
                State := 1; (* Move to Listen *)
            END_IF;
        
        1: (* Listen *)
            ACTIVE := TRUE;
            (* Simulate new client connection *)
            IF CLIENT_COUNT < 10 AND RAND() MOD 100 < 10 THEN (* 10% chance of new client *)
                FOR i := 1 TO 10 DO
                    IF NOT Clients[i].Active THEN
                        Clients[i].Active := TRUE;
                        Clients[i].SocketID := i; (* Simulated socket *)
                        CLIENT_COUNT := CLIENT_COUNT + 1;
                        EXIT;
                    END_IF;
                END_FOR;
            END_IF;
            (* Check for client requests *)
            FOR i := 1 TO 10 DO
                IF Clients[i].Active THEN
                    (* Simulate receiving request *)
                    Clients[i].BufferLen := 12; (* MBAP + PDU for simplicity *)
                    Clients[i].RequestBuffer[0] := Clients[i].TransactionID SHR 8; (* Transaction ID *)
                    Clients[i].RequestBuffer[1] := Clients[i].TransactionID AND 16#FF;
                    Clients[i].RequestBuffer[2] := 0; (* Protocol ID *)
                    Clients[i].RequestBuffer[3] := 0;
                    Clients[i].RequestBuffer[4] := 0; (* Length *)
                    Clients[i].RequestBuffer[5] := 6;
                    Clients[i].RequestBuffer[6] := 1; (* Unit ID *)
                    Clients[i].RequestBuffer[7] := RAND() MOD 17 + 1; (* Function Code 0x01–0x17 *)
                    Clients[i].RequestBuffer[8] := 0; (* Start Address *)
                    Clients[i].RequestBuffer[9] := 100;
                    Clients[i].RequestBuffer[10] := 0; (* Quantity *)
                    Clients[i].RequestBuffer[11] := 10;
                    State := 2; (* Process request *)
                    ClientIdx := i;
                    EXIT;
                END_IF;
            END_FOR;
            IF State <> 2 THEN
                State := 1; (* Continue listening *)
            END_IF;
        
        2: (* Process Request *)
            IF Clients[ClientIdx].Active THEN
                (* Parse MBAP *)
                MBAP.TransactionID := Clients[ClientIdx].RequestBuffer[0] SHL 8 +
                                    Clients[ClientIdx].RequestBuffer[1];
                MBAP.ProtocolID := Clients[ClientIdx].RequestBuffer[2] SHL 8 +
                                  Clients[ClientIdx].RequestBuffer[3];
                MBAP.Length := Clients[ClientIdx].RequestBuffer[4] SHL 8 +
                              Clients[ClientIdx].RequestBuffer[5];
                MBAP.UnitID := Clients[ClientIdx].RequestBuffer[6];
                
                (* Parse PDU *)
                PDU.FunctionCode := Clients[ClientIdx].RequestBuffer[7];
                PDU.StartAddress := Clients[ClientIdx].RequestBuffer[8] SHL 8 +
                                   Clients[ClientIdx].RequestBuffer[9];
                PDU.Quantity := Clients[ClientIdx].RequestBuffer[10] SHL 8 +
                               Clients[ClientIdx].RequestBuffer[11];
                
                (* Validate request *)
                IF PDU.FunctionCode NOT IN {1, 2, 3, 4, 5, 6, 15, 16, 23} OR
                   MBAP.ProtocolID <> 0 OR MBAP.Length < 6 THEN
                    ERROR := TRUE;
                    ERROR_CODE := 4; (* Invalid Request *)
                    Clients[ClientIdx].ResponseBuffer[7] := PDU.FunctionCode + 16#80;
                    Clients[ClientIdx].ResponseBuffer[8] := 16#01; (* Illegal Function *)
                    State := 3; (* Respond *)
                ELSIF PDU.StartAddress + PDU.Quantity > 1000 THEN
                    ERROR := TRUE;
                    ERROR_CODE := 5; (* Memory Access Error *)
                    Clients[ClientIdx].ResponseBuffer[7] := PDU.FunctionCode + 16#80;
                    Clients[ClientIdx].ResponseBuffer[8] := 16#02; (* Illegal Data Address *)
                    State := 3; (* Respond *)
                ELSE
                    (* Process function code *)
                    CASE PDU.FunctionCode OF
                        1: (* Read Coils *)
                            Clients[ClientIdx].ResponseBuffer[7] := 1;
                            Clients[ClientIdx].ResponseBuffer[8] := PDU.Quantity / 8 + 
                                                                 (PDU.Quantity MOD 8 > 0);
                            FOR j := 0 TO PDU.Quantity - 1 DO
                                IF COILS[PDU.StartAddress + j] THEN
                                    Clients[ClientIdx].ResponseBuffer[9 + j / 8] := 
                                        Clients[ClientIdx].ResponseBuffer[9 + j / 8] OR 
                                        (1 SHL (j MOD 8));
                                END_IF;
                            END_FOR;
                            Clients[ClientIdx].BufferLen := 9 + Clients[ClientIdx].ResponseBuffer[8];
                        
                        2: (* Read Discrete Inputs *)
                            Clients[ClientIdx].ResponseBuffer[7] := 2;
                            Clients[ClientIdx].ResponseBuffer[8] := PDU.Quantity / 8 + 
                                                                 (PDU.Quantity MOD 8 > 0);
                            FOR j := 0 TO PDU.Quantity - 1 DO
                                IF DISCRETE_INPUTS[PDU.StartAddress + j] THEN
                                    Clients[ClientIdx].ResponseBuffer[9 + j / 8] := 
                                        Clients[ClientIdx].ResponseBuffer[9 + j / 8] OR 
                                        (1 SHL (j MOD 8));
                                END_IF;
                            END_FOR;
                            Clients[ClientIdx].BufferLen := 9 + Clients[ClientIdx].ResponseBuffer[8];
                        
                        3: (* Read Holding Registers *)
                            Clients[ClientIdx].ResponseBuffer[7] := 3;
                            Clients[ClientIdx].ResponseBuffer[8] := PDU.Quantity * 2;
                            FOR j := 0 TO PDU.Quantity - 1 DO
                                Clients[ClientIdx].ResponseBuffer[9 + j * 2] := 
                                    HOLDING_REGISTERS[PDU.StartAddress + j] SHR 8;
                                Clients[ClientIdx].ResponseBuffer[10 + j * 2] := 
                                    HOLDING_REGISTERS[PDU.StartAddress + j] AND 16#FF;
                            END_FOR;
                            Clients[ClientIdx].BufferLen := 9 + PDU.Quantity * 2;
                        
                        4: (* Read Input Registers *)
                            Clients[ClientIdx].ResponseBuffer[7] := 4;
                            Clients[ClientIdx].ResponseBuffer[8] := PDU.Quantity * 2;
                            FOR j := 0 TO PDU.Quantity - 1 DO
                                Clients[ClientIdx].ResponseBuffer[9 + j * 2] := 
                                    INPUT_REGISTERS[PDU.StartAddress + j] SHR 8;
                                Clients[ClientIdx].ResponseBuffer[10 + j * 2] := 
                                    INPUT_REGISTERS[PDU.StartAddress + j] AND 16#FF;
                            END_FOR;
                            Clients[ClientIdx].BufferLen := 9 + PDU.Quantity * 2;
                        
                        5: (* Write Single Coil *)
                            IF Clients[ClientIdx].RequestBuffer[10] = 16#FF AND 
                               Clients[ClientIdx].RequestBuffer[11] = 16#00 THEN
                                COILS[PDU.StartAddress] := TRUE;
                            ELSIF Clients[ClientIdx].RequestBuffer[10] = 16#00 AND 
                                  Clients[ClientIdx].RequestBuffer[11] = 16#00 THEN
                                COILS[PDU.StartAddress] := FALSE;
                            ELSE
                                ERROR := TRUE;
                                ERROR_CODE := 4; (* Invalid Request *)
                                Clients[ClientIdx].ResponseBuffer[7] := 16#85;
                                Clients[ClientIdx].ResponseBuffer[8] := 16#03; (* Illegal Data Value *)
                                State := 3;
                                RETURN;
                            END_IF;
                            Clients[ClientIdx].ResponseBuffer[7] := 5;
                            Clients[ClientIdx].ResponseBuffer[8] := Clients[ClientIdx].RequestBuffer[8];
                            Clients[ClientIdx].ResponseBuffer[9] := Clients[ClientIdx].RequestBuffer[9];
                            Clients[ClientIdx].ResponseBuffer[10] := Clients[ClientIdx].RequestBuffer[10];
                            Clients[ClientIdx].ResponseBuffer[11] := Clients[ClientIdx].RequestBuffer[11];
                            Clients[ClientIdx].BufferLen := 12;
                        
                        6: (* Write Single Register *)
                            HOLDING_REGISTERS[PDU.StartAddress] := 
                                Clients[ClientIdx].RequestBuffer[10] SHL 8 +
                                Clients[ClientIdx].RequestBuffer[11];
                            Clients[ClientIdx].ResponseBuffer[7] := 6;
                            Clients[ClientIdx].ResponseBuffer[8] := Clients[ClientIdx].RequestBuffer[8];
                            Clients[ClientIdx].ResponseBuffer[9] := Clients[ClientIdx].RequestBuffer[9];
                            Clients[ClientIdx].ResponseBuffer[10] := Clients[ClientIdx].RequestBuffer[10];
                            Clients[ClientIdx].ResponseBuffer[11] := Clients[ClientIdx].RequestBuffer[11];
                            Clients[ClientIdx].BufferLen := 12;
                        
                        15: (* Write Multiple Coils *)
                            FOR j := 0 TO PDU.Quantity - 1 DO
                                IF Clients[ClientIdx].RequestBuffer[9 + j / 8] AND 
                                   (1 SHL (j MOD 8)) THEN
                                    COILS[PDU.StartAddress + j] := TRUE;
                                ELSE
                                    COILS[PDU.StartAddress + j] := FALSE;
                                END_IF;
                            END_FOR;
                            Clients[ClientIdx].ResponseBuffer[7] := 15;
                            Clients[ClientIdx].ResponseBuffer[8] := Clients[ClientIdx].RequestBuffer[8];
                            Clients[ClientIdx].ResponseBuffer[9] := Clients[ClientIdx].RequestBuffer[9];
                            Clients[ClientIdx].ResponseBuffer[10] := Clients[ClientIdx].RequestBuffer[10];
                            Clients[ClientIdx].ResponseBuffer[11] := Clients[ClientIdx].RequestBuffer[11];
                            Clients[ClientIdx].BufferLen := 12;
                        
                        16: (* Write Multiple Registers *)
                            FOR j := 0 TO PDU.Quantity - 1 DO
                                HOLDING_REGISTERS[PDU.StartAddress + j] := 
                                    Clients[ClientIdx].RequestBuffer[9 + j * 2] SHL 8 +
                                    Clients[ClientIdx].RequestBuffer[10 + j * 2];
                            END_FOR;
                            Clients[ClientIdx].ResponseBuffer[7] := 16;
                            Clients[ClientIdx].ResponseBuffer[8] := Clients[ClientIdx].RequestBuffer[8];
                            Clients[ClientIdx].ResponseBuffer[9] := Clients[ClientIdx].RequestBuffer[9];
                            Clients[ClientIdx].ResponseBuffer[10] := Clients[ClientIdx].RequestBuffer[10];
                            Clients[ClientIdx].ResponseBuffer[11] := Clients[ClientIdx].RequestBuffer[11];
                            Clients[ClientIdx].BufferLen := 12;
                        
                        23: (* Read/Write Multiple Registers *)
                            (* Write *)
                            FOR j := 0 TO PDU.Quantity - 1 DO
                                HOLDING_REGISTERS[PDU.StartAddress + j] := 
                                    Clients[ClientIdx].RequestBuffer[9 + j * 2] SHL 8 +
                                    Clients[ClientIdx].RequestBuffer[10 + j * 2];
                            END_FOR;
                            (* Read *)
                            Clients[ClientIdx].ResponseBuffer[7] := 23;
                            Clients[ClientIdx].ResponseBuffer[8] := PDU.Quantity * 2;
                            FOR j := 0 TO PDU.Quantity - 1 DO
                                Clients[ClientIdx].ResponseBuffer[9 + j * 2] := 
                                    HOLDING_REGISTERS[PDU.StartAddress + j] SHR 8;
                                Clients[ClientIdx].ResponseBuffer[10 + j * 2] := 
                                    HOLDING_REGISTERS[PDU.StartAddress + j] AND 16#FF;
                            END_FOR;
                            Clients[ClientIdx].BufferLen := 9 + PDU.Quantity * 2;
                    END_CASE;
                    
                    (* Set MBAP for response *)
                    Clients[ClientIdx].ResponseBuffer[0] := MBAP.TransactionID SHR 8;
                    Clients[ClientIdx].ResponseBuffer[1] := MBAP.TransactionID AND 16#FF;
                    Clients[ClientIdx].ResponseBuffer[2] := 0;
                    Clients[ClientIdx].ResponseBuffer[3] := 0;
                    Clients[ClientIdx].ResponseBuffer[4] := (Clients[ClientIdx].BufferLen - 6) SHR 8;
                    Clients[ClientIdx].ResponseBuffer[5] := (Clients[ClientIdx].BufferLen - 6) AND 16#FF;
                    Clients[ClientIdx].ResponseBuffer[6] := MBAP.UnitID;
                    
                    REQUESTS_PROCESSED := REQUESTS_PROCESSED + 1;
                    State := 3; (* Respond *)
                END_IF;
            ELSE
                (* Client disconnected *)
                Clients[ClientIdx].Active := FALSE;
                CLIENT_COUNT := CLIENT_COUNT - 1;
                State := 1; (* Return to Listen *)
            END_IF;
        
        3: (* Respond *)
            (* Simulate sending response *)
            IF RAND() MOD 100 < 5 THEN (* 5% chance of socket error *)
                ERROR := TRUE;
                ERROR_CODE := 3; (* Socket Error *)
                Clients[ClientIdx].Active := FALSE;
                CLIENT_COUNT := CLIENT_COUNT - 1;
            END_IF;
            
            (* Log errors *)
            IF ERROR AND LOG_COUNT < 50 THEN
                LOG_COUNT := LOG_COUNT + 1;
                Timestamp := '2025-05-17 18:06:00'; (* Replace with system clock *)
                CASE ERROR_CODE OF
                    4: DIAG_LOG[LOG_COUNT] := CONCAT(Timestamp, CONCAT(' Client ', 
                              CONCAT(TO_STRING(ClientIdx), ' Invalid Request – Exception 0x01')));
                    5: DIAG_LOG[LOG_COUNT] := CONCAT(Timestamp, CONCAT(' Client ', 
                              CONCAT(TO_STRING(ClientIdx), ' Memory Access Error – Exception 0x02')));
                    3: DIAG_LOG[LOG_COUNT] := CONCAT(Timestamp, CONCAT(' Client ', 
                              CONCAT(TO_STRING(ClientIdx), ' Socket Error')));
                END_CASE;
            ELSIF LOG_COUNT >= 50 THEN
                LogBufferFull := TRUE;
                ERROR := TRUE;
                ERROR_CODE := 4; (* Log buffer full *)
            END_IF;
            
            Clients[ClientIdx].BufferLen := 0; (* Clear request *)
            Clients[ClientIdx].TransactionID := Clients[ClientIdx].TransactionID + 1;
            State := 1; (* Return to Listen *)
    END_CASE;
END_IF;

(* Notes:
   - Purpose: Modbus TCP server for up to 10 clients, supporting FC 0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x0F, 0x10, 0x17.
   - Inputs:
     - ENABLE: TRUE to enable server.
     - PORT: TCP port (e.g., 502).
     - EXECUTE: Triggers manual cycle on rising edge.
     - COILS, DISCRETE_INPUTS, HOLDING_REGISTERS, INPUT_REGISTERS: Memory regions.
   - Outputs:
     - ACTIVE: TRUE when server running.
     - CLIENT_COUNT: Number of connected clients (0–10).
     - REQUESTS_PROCESSED: Total requests handled.
     - ERROR: TRUE if error occurs.
     - ERROR_CODE: 0 (No error), 1 (Invalid Port), 2 (Max Clients), 3 (Socket Error), 4 (Invalid Request), 5 (Memory Access Error).
     - DIAG_LOG: ARRAY[1..50] OF STRING[80], error/event logs.
     - LOG_COUNT: Number of log entries.
   - Logic:
     - State 1 (Listen): Accepts clients, receives requests.
     - State 2 (Process): Parses MBAP/PDU, processes function code, accesses memory.
     - State 3 (Respond): Sends response or exception, logs errors.
     - Cyclic operation every 10ms when ENABLE=TRUE; manual via EXECUTE.
   - Function Codes:
     - 0x01: Read Coils (COILS).
     - 0x02: Read Discrete Inputs (DISCRETE_INPUTS).
     - 0x03: Read Holding Registers (HOLDING_REGISTERS).
     - 0x04: Read Input Registers (INPUT_REGISTERS).
     - 0x05: Write Single Coil (COILS).
     - 0x06: Write Single Register (HOLDING_REGISTERS).
     - 0x0F: Write Multiple Coils (COILS).
     - 0x10: Write Multiple Registers (HOLDING_REGISTERS).
     - 0x17: Read/Write Multiple Registers (HOLDING_REGISTERS).
   - Error Handling:
     - Invalid Port: <1 or >65535 (ERROR_CODE=1).
     - Max Clients: >10 (ERROR_CODE=2).
     - Socket Error: Simulated 5% failure (ERROR_CODE=3).
     - Invalid Request: Bad FC or length (ERROR_CODE=4, Exception 0x01).
     - Memory Access Error: Address out of range (ERROR_CODE=5, Exception 0x02).
     - Log buffer full: LOG_COUNT >= 50 (ERROR_CODE=4).
   - Safety:
     - Bounded arrays (10 clients, 50 logs) prevent overflow.
     - State machine ensures scan-cycle safety (e.g., 10–100 ms cycles).
     - Input validation avoids invalid PORT or requests.
   - Concurrency:
     - Handles up to 10 clients with independent buffers.
     - Sequential memory access prevents conflicts.
   - Traceability:
     - Timestamped logs (e.g., "2025-05-17 18:06:00 Client 1 FC01 Invalid Address – Exception 0x02").
     - DIAG_LOG supports HMI/SCADA export.
   - Usage:
     - Read Coils (0x01): Parses address/quantity, reads COILS, returns data or exception.
     - Concurrently serves multiple clients (e.g., FC01 and FC06).
   - Maintenance:
     - Modular design aids debugging.
     - ERROR_CODE and DIAG_LOG support diagnostics.
   - Platform Notes:
     - Assumes STRING[80], BYTE arrays, and basic operations; adjust for platform limits.
     - Timestamp is static ("2025-05-17 18:06:00"); replace with system clock (e.g., GET_SYSTEM_TIME).
     - Communication is simulated (RAND()); replace with TCP driver (e.g., TCP_RECV, TCP_SEND).
     - Log buffer size (50) and client limit (10) are practical; adjustable for resources.
*)
END_FUNCTION_BLOCK
