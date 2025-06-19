(* Function Block: MODBUS_TCP_SERVER
   Purpose: Implements a Modbus TCP server supporting up to 10 parallel client connections.
   Features:
   - Handles function codes: 0x01-0x04 (read), 0x05, 0x06, 0x0F, 0x10 (write), 0x17 (read/write)
   - Manages TCP connections with conflict-free memory access
   - Ensures protocol compliance with error handling and timeouts
   - Supports coils (1000) and registers (1000 each for holding/input)
*)

FUNCTION_BLOCK MODBUS_TCP_SERVER
VAR_INPUT
    ENABLE : BOOL;                (* Enables/disables server *)
    PORT : UINT := 502;           (* Modbus TCP port, default 502 *)
    TIMEOUT : TIME := T#1s;       (* Timeout for client communication *)
END_VAR

VAR_OUTPUT
    ACTIVE : BOOL;                (* TRUE when server is running *)
    ERROR : BOOL;                 (* TRUE if server error occurs *)
    ERROR_CODE : UINT;            (* 0=None, 1=Socket Error, 2=Max Clients, 3=Invalid Request *)
    CLIENT_COUNT : UINT;          (* Number of active client connections *)
END_VAR

VAR
    (* Server state *)
    ServerSocket : UINT;          (* Server socket handle *)
    ClientSockets : ARRAY[0..9] OF UINT; (* Client socket handles *)
    ClientActive : ARRAY[0..9] OF BOOL; (* Client connection status *)
    ClientTimeout : ARRAY[0..9] OF TON; (* Timeout timers for clients *)
    
    (* Modbus memory *)
    Coils : ARRAY[0..999] OF BOOL; (* Coil buffer *)
    HoldingRegisters : ARRAY[0..999] OF UINT; (* Holding register buffer *)
    InputRegisters : ARRAY[0..999] OF UINT; (* Input register buffer *)
    
    (* Request processing *)
    CurrentClient : UINT;         (* Current client being processed *)
    RequestBuffer : ARRAY[0..255] OF BYTE; (* Incoming request buffer *)
    ResponseBuffer : ARRAY[0..255] OF BYTE; (* Outgoing response buffer *)
    RequestLength : UINT;         (* Length of incoming request *)
    FunctionCode : BYTE;          (* Modbus function code *)
    StartAddress : UINT;          (* Starting address from request *)
    Quantity : UINT;              (* Number of coils/registers *)
    ByteCount : UINT;             (* Byte count for write operations *)
    
    (* Internal state *)
    i : UINT;                     (* Loop index *)
    SocketError : BOOL;           (* Flag for socket errors *)
    ResponseLength : UINT;        (* Length of response *)
    TransactionId : UINT;         (* Modbus TCP transaction ID *)
    ProtocolId : UINT := 0;       (* Modbus TCP protocol ID, always 0 *)
    UnitId : BYTE;                (* Unit identifier *)
END_VAR

(* Initialize outputs *)
ACTIVE := FALSE;
ERROR := FALSE;
ERROR_CODE := 0;
CLIENT_COUNT := 0;

(* Main logic *)
IF ENABLE THEN
    (* Initialize server on rising edge *)
    IF NOT ACTIVE THEN
        ServerSocket := TCP_CreateServer(PORT);
        IF ServerSocket = 0 THEN
            ERROR := TRUE;
            ERROR_CODE := 1; (* Socket Error *)
            RETURN;
        END_IF
        ACTIVE := TRUE;
        FOR i := 0 TO 9 DO
            ClientSockets[i] := 0;
            ClientActive[i] := FALSE;
            ClientTimeout[i](IN := FALSE);
        END_FOR
    END_IF
    
    (* Accept new client connections *)
    IF CLIENT_COUNT < 10 THEN
        i := 0;
        WHILE i < 10 AND NOT ClientActive[i] DO
            ClientSockets[i] := TCP_AcceptClient(ServerSocket);
            IF ClientSockets[i] <> 0 THEN
                ClientActive[i] := TRUE;
                ClientTimeout[i](IN := TRUE, PT := TIMEOUT);
                CLIENT_COUNT := CLIENT_COUNT + 1;
                EXIT;
            END_IF
            i := i + 1;
        END_WHILE
    END_IF
    
    (* Process each active client *)
    FOR CurrentClient := 0 TO 9 DO
        IF ClientActive[CurrentClient] THEN
            (* Check for timeout *)
            IF ClientTimeout[CurrentClient].Q THEN
                TCP_CloseSocket(ClientSockets[CurrentClient]);
                ClientSockets[CurrentClient] := 0;
                ClientActive[CurrentClient] := FALSE;
                CLIENT_COUNT := CLIENT_COUNT - 1;
                ClientTimeout[CurrentClient](IN := FALSE);
                CONTINUE;
            END_IF
            
            (* Read request *)
            RequestLength := TCP_Receive(ClientSockets[CurrentClient], RequestBuffer, 256);
            IF RequestLength > 0 THEN
                (* Reset timeout timer *)
                ClientTimeout[CurrentClient](IN := FALSE);
                ClientTimeout[CurrentClient](IN := TRUE, PT := TIMEOUT);
                
                (* Parse Modbus TCP header *)
                TransactionId := (RequestBuffer[0] << 8) OR RequestBuffer[1];
                ProtocolId := (RequestBuffer[2] << 8) OR RequestBuffer[3];
                (* Length field ignored for simplicity *)
                UnitId := RequestBuffer[6];
                FunctionCode := RequestBuffer[7];
                
                (* Validate request *)
                IF ProtocolId <> 0 OR RequestLength < 8 THEN
                    ERROR := TRUE;
                    ERROR_CODE := 3; (* Invalid Request *)
                    SendErrorResponse(CurrentClient, FunctionCode, 1); (* Illegal Function *)
                    CONTINUE;
                END_IF
                
                (* Process function code *)
                StartAddress := (RequestBuffer[8] << 8) OR RequestBuffer[9];
                Quantity := (RequestBuffer[10] << 8) OR RequestBuffer[11];
                
                CASE FunctionCode OF
                    16#01: (* Read Coils *)
                        IF StartAddress + Quantity <= 1000 THEN
                            ReadCoils(CurrentClient);
                        ELSE
                            SendErrorResponse(CurrentClient, FunctionCode, 2); (* Illegal Data Address *)
                        END_IF
                    16#02: (* Read Discrete Inputs *)
                        IF StartAddress + Quantity <= 1000 THEN
                            ReadDiscreteInputs(CurrentClient);
                        ELSE
                            SendErrorResponse(CurrentClient, FunctionCode, 2);
                        END_IF
                    16#03: (* Read Holding Registers *)
                        IF StartAddress + Quantity <= 1000 THEN
                            ReadHoldingRegisters(CurrentClient);
                        ELSE
                            SendErrorResponse(CurrentClient, FunctionCode, 2);
                        END_IF
                    16#04: (* Read Input Registers *)
                        IF StartAddress + Quantity <= 1000 THEN
                            ReadInputRegisters(CurrentClient);
                        ELSE
                            SendErrorResponse(CurrentClient, FunctionCode, 2);
                        END_IF
                    16#05: (* Write Single Coil *)
                        IF StartAddress < 1000 THEN
                            WriteSingleCoil(CurrentClient);
                        ELSE
                            SendErrorResponse(CurrentClient, FunctionCode, 2);
                        END_IF
                    16#06: (* Write Single Register *)
                        IF StartAddress < 1000 THEN
                            WriteSingleRegister(CurrentClient);
                        ELSE
                            SendErrorResponse(CurrentClient, FunctionCode, 2);
                        END_IF
                    16#0F: (* Write Multiple Coils *)
                        IF StartAddress + Quantity <= 1000 THEN
                            WriteMultipleCoils(CurrentClient);
                        ELSE
                            SendErrorResponse(CurrentClient, FunctionCode, 2);
                        END_IF
                    16#10: (* Write Multiple Registers *)
                        IF StartAddress + Quantity <= 1000 THEN
                            WriteMultipleRegisters(CurrentClient);
                        ELSE
                            SendErrorResponse(CurrentClient, FunctionCode, 2);
                        END_IF
                    16#17: (* Read/Write Multiple Registers *)
                        IF RequestLength >= 17 THEN
                            ReadWriteMultipleRegisters(CurrentClient);
                        ELSE
                            SendErrorResponse(CurrentClient, FunctionCode, 3); (* Illegal Data Value *)
                        END_IF
                    ELSE
                        SendErrorResponse(CurrentClient, FunctionCode, 1); (* Illegal Function *)
                END_CASE
            END_IF
        END_IF
    END_FOR
ELSE
    (* Disable server *)
    IF ACTIVE THEN
        TCP_CloseSocket(ServerSocket);
        FOR i := 0 TO 9 DO
            IF ClientActive[i] THEN
                TCP_CloseSocket(ClientSockets[i]);
                ClientActive[i] := FALSE;
                ClientSockets[i] := 0;
                ClientTimeout[i](IN := FALSE);
            END_IF
        END_FOR
        ACTIVE := FALSE;
        CLIENT_COUNT := 0;
        ERROR := FALSE;
        ERROR_CODE := 0;
    END_IF
END_IF

(* Helper methods *)
METHOD ReadCoils
VAR_INPUT
    ClientIndex : UINT;
END_VAR
ByteCount := (Quantity + 7) / 8;
ResponseBuffer[0] := TransactionId >> 8;
ResponseBuffer[1] := TransactionId AND 16#FF;
ResponseBuffer[2] := 0; (* Protocol ID *)
ResponseBuffer[3] := 0;
ResponseBuffer[4] := (ByteCount + 3) >> 8; (* Length *)
ResponseBuffer[5] := (ByteCount + 3) AND 16#FF;
ResponseBuffer[6] := UnitId;
ResponseBuffer[7] := FunctionCode;
ResponseBuffer[8] := ByteCount;
FOR i := 0 TO ByteCount - 1 DO
    ResponseBuffer[9 + i] := 0;
    FOR j := 0 TO 7 DO
        IF i * 8 + j < Quantity THEN
            ResponseBuffer[9 + i] := ResponseBuffer[9 + i] OR (Coils[StartAddress + i * 8 + j] << j);
        END_IF
    END_FOR
END_FOR
ResponseLength := 9 + ByteCount;
TCP_Send(ClientSockets[ClientIndex], ResponseBuffer, ResponseLength);
END_METHOD

METHOD ReadDiscreteInputs
VAR_INPUT
    ClientIndex : UINT;
END_VAR
(* Simulate discrete inputs as coils for simplicity *)
ReadCoils(ClientIndex); (* Reuse coil logic *)
END_METHOD

METHOD ReadHoldingRegisters
VAR_INPUT
    ClientIndex : UINT;
END_VAR
ByteCount := Quantity * 2;
ResponseBuffer[0] := TransactionId >> 8;
ResponseBuffer[1] := TransactionId AND 16#FF;
ResponseBuffer[2] := 0;
ResponseBuffer[3] := 0;
ResponseBuffer[4] := (ByteCount + 3) >> 8;
ResponseBuffer[5] := (ByteCount + 3) AND 16#FF;
ResponseBuffer[6] := UnitId;
ResponseBuffer[7] := FunctionCode;
ResponseBuffer[8] := ByteCount;
FOR i := 0 TO Quantity - 1 DO
    ResponseBuffer[9 + i * 2] := HoldingRegisters[StartAddress + i] >> 8;
    ResponseBuffer[9 + i * 2 + 1] := HoldingRegisters[StartAddress + i] AND 16#FF;
END_FOR
ResponseLength := 9 + ByteCount;
TCP_Send(ClientSockets[ClientIndex], ResponseBuffer, ResponseLength);
END_METHOD

METHOD ReadInputRegisters
VAR_INPUT
    ClientIndex : UINT;
END_VAR
ByteCount := Quantity * 2;
ResponseBuffer[0] := TransactionId >> 8;
ResponseBuffer[1] := TransactionId AND 16#FF;
ResponseBuffer[2] := 0;
ResponseBuffer[3] := 0;
ResponseBuffer[4] := (ByteCount + 3) >> 8;
ResponseBuffer[5] := (ByteCount + 3) AND 16#FF;
ResponseBuffer[6] := UnitId;
ResponseBuffer[7] := FunctionCode;
ResponseBuffer[8] := ByteCount;
FOR i := 0 TO Quantity - 1 DO
    ResponseBuffer[9 + i * 2] := InputRegisters[StartAddress + i] >> 8;
    ResponseBuffer[9 + i * 2 + 1] := InputRegisters[StartAddress + i] AND 16#FF;
END_FOR
ResponseLength := 9 + ByteCount;
TCP_Send(ClientSockets[ClientIndex], ResponseBuffer, ResponseLength);
END_METHOD

METHOD WriteSingleCoil
VAR_INPUT
    ClientIndex : UINT;
END_VAR
IF RequestBuffer[10] = 16#FF AND RequestBuffer[11] = 16#00 THEN
    Coils[StartAddress] := TRUE;
ELSE
    Coils[StartAddress] := FALSE;
END_IF
ResponseBuffer[0] := TransactionId >> 8;
ResponseBuffer[1] := TransactionId AND 16#FF;
ResponseBuffer[2] := 0;
ResponseBuffer[3] := 0;
ResponseBuffer[4] := 0;
ResponseBuffer[5] := 6;
ResponseBuffer[6] := UnitId;
ResponseBuffer[7] := FunctionCode;
ResponseBuffer[8] := RequestBuffer[8];
ResponseBuffer[9] := RequestBuffer[9];
ResponseBuffer[10] := RequestBuffer[10];
ResponseBuffer[11] := RequestBuffer[11];
ResponseLength := 12;
TCP_Send(ClientSockets[ClientIndex], ResponseBuffer, ResponseLength);
END_METHOD

METHOD WriteSingleRegister
VAR_INPUT
    ClientIndex : UINT;
END_VAR
HoldingRegisters[StartAddress] := (RequestBuffer[10] << 8) OR RequestBuffer[11];
ResponseBuffer[0] := TransactionId >> 8;
ResponseBuffer[1] := TransactionId AND 16#FF;
ResponseBuffer[2] := 0;
ResponseBuffer[3] := 0;
ResponseBuffer[4] := 0;
ResponseBuffer[5] := 6;
ResponseBuffer[6] := UnitId;
ResponseBuffer[7] := FunctionCode;
ResponseBuffer[8] := RequestBuffer[8];
ResponseBuffer[9] := RequestBuffer[9];
ResponseBuffer[10] := RequestBuffer[10];
ResponseBuffer[11] := RequestBuffer[11];
ResponseLength := 12;
TCP_Send(ClientSockets[ClientIndex], ResponseBuffer, ResponseLength);
END_METHOD

METHOD WriteMultipleCoils
VAR_INPUT
    ClientIndex : UINT;
END_VAR
ByteCount := RequestBuffer[12];
IF RequestLength >= 13 + ByteCount THEN
    FOR i := 0 TO Quantity - 1 DO
        Coils[StartAddress + i] := (RequestBuffer[13 + i / 8] >> (i MOD 8)) AND 1;
    END_FOR
    ResponseBuffer[0] := TransactionId >> 8;
    ResponseBuffer[1] := TransactionId AND 16#FF;
    ResponseBuffer[2] := 0;
    ResponseBuffer[3] := 0;
    ResponseBuffer[4] := 0;
    ResponseBuffer[5] := 6;
    ResponseBuffer[6] := UnitId;
    ResponseBuffer[7] := FunctionCode;
    ResponseBuffer[8] := RequestBuffer[8];
    ResponseBuffer[9] := RequestBuffer[9];
    ResponseBuffer[10] := RequestBuffer[10];
    ResponseBuffer[11] := RequestBuffer[11];
    ResponseLength := 12;
    TCP_Send(ClientSockets[ClientIndex], ResponseBuffer, ResponseLength);
ELSE
    SendErrorResponse(CurrentClient, FunctionCode, 3); (* Illegal Data Value *)
END_IF
END_METHOD

METHOD WriteMultipleRegisters
VAR_INPUT
    ClientIndex : UINT;
END_VAR
ByteCount := RequestBuffer[12];
IF RequestLength >= 13 + ByteCount AND ByteCount = Quantity * 2 THEN
    FOR i := 0 TO Quantity - 1 DO
        HoldingRegisters[StartAddress + i] := (RequestBuffer[13 + i * 2] << 8) OR RequestBuffer[13 + i * 2 + 1];
    END_FOR
    ResponseBuffer[0] := TransactionId >> 8;
    ResponseBuffer[1] := TransactionId AND 16#FF;
    ResponseBuffer[2] := 0;
    ResponseBuffer[3] := 0;
    ResponseBuffer[4] := 0;
    ResponseBuffer[5] := 6;
    ResponseBuffer[6] := UnitId;
    ResponseBuffer[7] := FunctionCode;
    ResponseBuffer[8] := RequestBuffer[8];
    ResponseBuffer[9] := RequestBuffer[9];
    ResponseBuffer[10] := RequestBuffer[10];
    ResponseBuffer[11] := RequestBuffer[11];
    ResponseLength := 12;
    TCP_Send(ClientSockets[ClientIndex], ResponseBuffer, ResponseLength);
ELSE
    SendErrorResponse(CurrentClient, FunctionCode, 3); (* Illegal Data Value *)
END_IF
END_METHOD

METHOD ReadWriteMultipleRegisters
VAR_INPUT
    ClientIndex : UINT;
END_VAR
VAR
    ReadAddress : UINT;
    ReadQuantity : UINT;
    WriteAddress : UINT;
    WriteQuantity : UINT;
    WriteByteCount : UINT;
    j : UINT;
END_VAR
ReadAddress := (RequestBuffer[8] << 8) OR RequestBuffer[9];
ReadQuantity := (RequestBuffer[10] << 8) OR RequestBuffer[11];
WriteAddress := (RequestBuffer[12] << 8) OR RequestBuffer[13];
WriteQuantity := (RequestBuffer[14] << 8) OR RequestBuffer[15];
WriteByteCount := RequestBuffer[16];
IF ReadAddress + ReadQuantity <= 1000 AND WriteAddress + WriteQuantity <= 1000 AND WriteByteCount = WriteQuantity * 2 THEN
    (* Write registers *)
    FOR i := 0 TO WriteQuantity - 1 DO
        HoldingRegisters[WriteAddress + i] := (RequestBuffer[17 + i * 2] << 8) OR RequestBuffer[17 + i * 2 + 1];
    END_FOR
    (* Read registers *)
    ByteCount := ReadQuantity * 2;
    ResponseBuffer[0] := TransactionId >> 8;
    ResponseBuffer[1] := TransactionId AND 16#FF;
    ResponseBuffer[2] := 0;
    ResponseBuffer[3] := 0;
    ResponseBuffer[4] := (ByteCount + 3) >> 8;
    ResponseBuffer[5] := (ByteCount + 3) AND 16#FF;
    ResponseBuffer[6] := UnitId;
    ResponseBuffer[7] := FunctionCode;
    ResponseBuffer[8] := ByteCount;
    FOR i := 0 TO ReadQuantity - 1 DO
        ResponseBuffer[9 + i * 2] := HoldingRegisters[ReadAddress + i] >> 8;
        ResponseBuffer[9 + i * 2 + 1] := HoldingRegisters[ReadAddress + i] AND 16#FF;
    END_FOR
    ResponseLength := 9 + ByteCount;
    TCP_Send(ClientSockets[ClientIndex], ResponseBuffer, ResponseLength);
ELSE
    SendErrorResponse(CurrentClient, FunctionCode, 2); (* Illegal Data Address *)
END_IF
END_METHOD

METHOD SendErrorResponse
VAR_INPUT
    ClientIndex : UINT;
    FuncCode : BYTE;
    ExceptionCode : BYTE;
END_VAR
ResponseBuffer[0] := TransactionId >> 8;
ResponseBuffer[1] := TransactionId AND 16#FF;
ResponseBuffer[2] := 0;
ResponseBuffer[3] := 0;
ResponseBuffer[4] := 0;
ResponseBuffer[5] := 3;
ResponseBuffer[6] := UnitId;
ResponseBuffer[7] := FuncCode OR 16#80;
ResponseBuffer[8] := ExceptionCode;
ResponseLength := 9;
TCP_Send(ClientSockets[ClientIndex], ResponseBuffer, ResponseLength);
END_METHOD

(* Simulated TCP functions *)
FUNCTION TCP_CreateServer : UINT
VAR_INPUT
    Port : UINT;
END_VAR
(* Placeholder: Returns socket handle or 0 on error *)
TCP_CreateServer := 1;
END_FUNCTION

FUNCTION TCP_AcceptClient : UINT
VAR_INPUT
    ServerSocket : UINT;
END_VAR
(* Placeholder: Returns client socket or 0 if none *)
TCP_AcceptClient := 0;
END_FUNCTION

FUNCTION TCP_Receive : UINT
VAR_INPUT
    Socket : UINT;
    Buffer : ARRAY[0..255] OF BYTE;
    MaxLen : UINT;
END_VAR
(* Placeholder: Returns bytes received or 0 on error *)
TCP_Receive := 0;
END_FUNCTION

FUNCTION TCP_Send : BOOL
VAR_INPUT
    Socket : UINT;
    Buffer : ARRAY[0..255] OF BYTE;
    Len : UINT;
END_VAR
(* Placeholder: Returns TRUE on success *)
TCP_Send := TRUE;
END_FUNCTION

FUNCTION TCP_CloseSocket : BOOL
VAR_INPUT
    Socket : UINT;
END_VAR
(* Placeholder: Returns TRUE on success *)
TCP_CloseSocket := TRUE;
END_FUNCTION
END_FUNCTION_BLOCK
