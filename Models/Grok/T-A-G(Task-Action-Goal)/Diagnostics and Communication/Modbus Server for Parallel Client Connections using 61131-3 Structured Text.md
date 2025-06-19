(* Function Block: Modbus TCP Server *)
FUNCTION_BLOCK FB_ModbusTCPServer
VAR_INPUT
    Enable : BOOL;                     (* Enable server *)
    LocalIP : STRING[15];              (* Local IP address *)
    Port : UINT := 502;                (* Modbus TCP port, default 502 *)
    TimeoutMs : UINT := 5000;          (* Connection timeout in ms *)
END_VAR

VAR_OUTPUT
    Active : BOOL;                     (* Server is active *)
    Error : BOOL;                      (* Error flag *)
    ErrorID : UINT;                    (* Error code *)
    ConnectedClients : UINT;           (* Number of connected clients *)
END_VAR

VAR
    ServerHandle : DWORD;              (* Server socket handle *)
    ClientHandles : ARRAY[0..9] OF DWORD; (* Client connection handles *)
    ClientStates : ARRAY[0..9] OF UINT; (* Client states: 0=Idle, 1=Connected, 2=Processing *)
    RxBuffer : ARRAY[0..9, 0..255] OF BYTE; (* Receive buffers for clients *)
    TxBuffer : ARRAY[0..9, 0..255] OF BYTE; (* Transmit buffers for clients *)
    RxLength : ARRAY[0..9] OF UINT;    (* Received data length *)
    Coils : ARRAY[0..9999] OF BOOL;    (* Coil data buffer *)
    DiscreteInputs : ARRAY[0..9999] OF BOOL; (* Discrete inputs buffer *)
    HoldingRegisters : ARRAY[0..9999] OF UINT; (* Holding registers buffer *)
    InputRegisters : ARRAY[0..9999] OF UINT; (* Input registers buffer *)
    i, j : UINT;                       (* Loop counters *)
    MBAP : ARRAY[0..6] OF BYTE;        (* Modbus Application Header *)
    TransactionID : UINT;              (* Modbus transaction ID *)
    LastError : UINT;                  (* Last error code *)
END_VAR

(* Initialize outputs *)
Active := FALSE;
Error := FALSE;
ErrorID := 0;
ConnectedClients := 0;

(* Main logic *)
IF Enable THEN
    Active := TRUE;
    
    (* Initialize server if not already running *)
    IF ServerHandle = 0 THEN
        ServerHandle := SysSocketCreate(LocalIP, Port, 10);
        IF ServerHandle = 0 THEN
            Error := TRUE;
            ErrorID := 100; (* Server creation failed *)
            Active := FALSE;
            RETURN;
        END_IF;
    END_IF;

    (* Handle client connections *)
    FOR i := 0 TO 9 DO
        CASE ClientStates[i] OF
            0: (* Idle *)
                ClientHandles[i] := SysSocketAccept(ServerHandle, TimeoutMs);
                IF ClientHandles[i] <> 0 THEN
                    ClientStates[i] := 1; (* Connected *)
                    ConnectedClients := ConnectedClients + 1;
                    RxLength[i] := 0;
                END_IF;

            1: (* Connected *)
                (* Receive data *)
                RxLength[i] := SysSocketReceive(ClientHandles[i], RxBuffer[i], 256, TimeoutMs);
                IF RxLength[i] > 7 THEN (* Minimum Modbus TCP frame size *)
                    ClientStates[i] := 2; (* Process request *)
                ELSIF RxLength[i] = 0 THEN
                    (* Client disconnected *)
                    SysSocketClose(ClientHandles[i]);
                    ClientHandles[i] := 0;
                    ClientStates[i] := 0;
                    ConnectedClients := ConnectedClients - 1;
                ELSIF LastError = 101 THEN
                    (* Timeout *)
                    Error := TRUE;
                    ErrorID := 101; (* Client timeout *)
                    SysSocketClose(ClientHandles[i]);
                    ClientHandles[i] := 0;
                    ClientStates[i] := 0;
                    ConnectedClients := ConnectedClients - 1;
                END_IF;

            2: (* Processing *)
                ProcessModbusRequest(i);
                ClientStates[i] := 1; (* Return to Connected *)
        END_CASE;
    END_FOR;

ELSE
    (* Disable server *)
    IF Active THEN
        FOR i := 0 TO 9 DO
            IF ClientHandles[i] <> 0 THEN
                SysSocketClose(ClientHandles[i]);
                ClientHandles[i] := 0;
                ClientStates[i] := 0;
            END_IF;
        END_FOR;
        SysSocketClose(ServerHandle);
        ServerHandle := 0;
        Active := FALSE;
        ConnectedClients := 0;
        Error := FALSE;
        ErrorID := 0;
    END_IF;
END_IF;

(* Process Modbus request *)
METHOD ProcessModbusRequest
VAR_INPUT
    ClientIndex : UINT;
END_VAR
VAR
    FunctionCode : BYTE;
    StartAddress : UINT;
    Quantity : UINT;
    ByteCount : UINT;
    ResponseLength : UINT;
    k : UINT;
END_VAR

(* Parse MBAP header *)
TransactionID := BYTES_TO_UINT(RxBuffer[ClientIndex, 0], RxBuffer[ClientIndex, 1]);
MBAP[0] := RxBuffer[ClientIndex, 0]; (* Transaction ID *)
MBAP[1] := RxBuffer[ClientIndex, 1];
MBAP[2] := RxBuffer[ClientIndex, 2]; (* Protocol ID *)
MBAP[3] := RxBuffer[ClientIndex, 3];
MBAP[4] := 0; (* Length placeholder *)
MBAP[5] := 0;
MBAP[6] := RxBuffer[ClientIndex, 6]; (* Unit ID *)

FunctionCode := RxBuffer[ClientIndex, 7];
ResponseLength := 0;

CASE FunctionCode OF
    1: (* Read Coils *)
        StartAddress := BYTES_TO_UINT(RxBuffer[ClientIndex, 8], RxBuffer[ClientIndex, 9]);
        Quantity := BYTES_TO_UINT(RxBuffer[ClientIndex, 10], RxBuffer[ClientIndex, 11]);
        
        IF StartAddress + Quantity <= 10000 THEN
            (* Prepare response *)
            TxBuffer[ClientIndex, 0] := MBAP[0];
            TxBuffer[ClientIndex, 1] := MBAP[1];
            TxBuffer[ClientIndex, 2] := MBAP[2];
            TxBuffer[ClientIndex, 3] := MBAP[3];
            ByteCount := (Quantity + 7) / 8;
            TxBuffer[ClientIndex, 4] := HIGH_BYTE(3 + ByteCount);
            TxBuffer[ClientIndex, 5] := LOW_BYTE(3 + ByteCount);
            TxBuffer[ClientIndex, 6] := MBAP[6];
            TxBuffer[ClientIndex, 7] := FunctionCode;
            TxBuffer[ClientIndex, 8] := ByteCount;
            
            (* Pack coils into bytes *)
            FOR k := 0 TO ByteCount - 1 DO
                TxBuffer[ClientIndex, 9 + k] := 0;
                FOR j := 0 TO 7 DO
                    IF (k * 8 + j) < Quantity THEN
                        TxBuffer[ClientIndex, 9 + k] := TxBuffer[ClientIndex, 9 + k] OR (BOOL_TO_BYTE(Coils[StartAddress + k * 8 + j]) << j);
                    END_IF;
                END_FOR;
            END_FOR;
            ResponseLength := 9 + ByteCount;
        ELSE
            SendException(ClientIndex, FunctionCode, 2); (* Illegal data address *)
            RETURN;
        END_IF;

    2: (* Read Discrete Inputs *)
        (* Similar to Read Coils, using DiscreteInputs buffer *)
        StartAddress := BYTES_TO_UINT(RxBuffer[ClientIndex, 8], RxBuffer[ClientIndex, 9]);
        Quantity := BYTES_TO_UINT(RxBuffer[ClientIndex, 10], RxBuffer[ClientIndex, 11]);
        IF StartAddress + Quantity <= 10000 THEN
            TxBuffer[ClientIndex, 0] := MBAP[0];
            TxBuffer[ClientIndex, 1] := MBAP[1];
            TxBuffer[ClientIndex, 2] := MBAP[2];
            TxBuffer[ClientIndex, 3] := MBAP[3];
            ByteCount := (Quantity + 7) / 8;
            TxBuffer[ClientIndex, 4] := HIGH_BYTE(3 + ByteCount);
            TxBuffer[ClientIndex, 5] := LOW_BYTE(3 + ByteCount);
            TxBuffer[ClientIndex, 6] := MBAP[6];
            TxBuffer[ClientIndex, 7] := FunctionCode;
            TxBuffer[ClientIndex, 8] := ByteCount;
            FOR k := 0 TO ByteCount - 1 DO
                TxBuffer[ClientIndex, 9 + k] := 0;
                FOR j := 0 TO 7 DO
                    IF (k * 8 + j) < Quantity THEN
                        TxBuffer[ClientIndex, 9 + k] := TxBuffer[ClientIndex, 9 + k] OR (BOOL_TO_BYTE(DiscreteInputs[StartAddress + k * 8 + j]) << j);
                    END_IF;
                END_FOR;
            END_FOR;
            ResponseLength := 9 + ByteCount;
        ELSE
            SendException(ClientIndex, FunctionCode, 2);
            RETURN;
        END_IF;

    3: (* Read Holding Registers *)
        StartAddress := BYTES_TO_UINT(RxBuffer[ClientIndex, 8], RxBuffer[ClientIndex, 9]);
        Quantity := BYTES_TO_UINT(RxBuffer[ClientIndex, 10], RxBuffer[ClientIndex, 11]);
        IF StartAddress + Quantity <= 10000 THEN
            TxBuffer[ClientIndex, 0] := MBAP[0];
            TxBuffer[ClientIndex, 1] := MBAP[1];
            TxBuffer[ClientIndex, 2] := MBAP[2];
            TxBuffer[ClientIndex, 3] := MBAP[3];
            ByteCount := Quantity * 2;
            TxBuffer[ClientIndex, 4] := HIGH_BYTE(3 + ByteCount);
            TxBuffer[ClientIndex, 5] := LOW_BYTE(3 + ByteCount);
            TxBuffer[ClientIndex, 6] := MBAP[6];
            TxBuffer[ClientIndex, 7] := FunctionCode;
            TxBuffer[ClientIndex, 8] := ByteCount;
            FOR k := 0 TO Quantity - 1 DO
                TxBuffer[ClientIndex, 9 + k * 2] := HIGH_BYTE(HoldingRegisters[StartAddress + k]);
                TxBuffer[ClientIndex, 10 + k * 2] := LOW_BYTE(HoldingRegisters[StartAddress + k]);
            END_FOR;
            ResponseLength := 9 + ByteCount;
        ELSE
            SendException(ClientIndex, FunctionCode, 2);
            RETURN;
        END_IF;

    4: (* Read Input Registers *)
        (* Similar to Read Holding Registers, using InputRegisters *)
        StartAddress := BYTES_TO_UINT(RxBuffer[ClientIndex, 8], RxBuffer[ClientIndex, 9]);
        Quantity := BYTES_TO_UINT(RxBuffer[ClientIndex, 10], RxBuffer[ClientIndex, 11]);
        IF StartAddress + Quantity <= 10000 THEN
            TxBuffer[ClientIndex, 0] := MBAP[0];
            TxBuffer[ClientIndex, 1] := MBAP[1];
            TxBuffer[ClientIndex, 2] := MBAP[2];
            TxBuffer[ClientIndex, 3] := MBAP[3];
            ByteCount := Quantity * 2;
            TxBuffer[ClientIndex, 4] := HIGH_BYTE(3 + ByteCount);
            TxBuffer[ClientIndex, 5] := LOW_BYTE(3 + ByteCount);
            TxBuffer[ClientIndex, 6] := MBAP[6];
            TxBuffer[ClientIndex, 7] := FunctionCode;
            TxBuffer[ClientIndex, 8] := ByteCount;
            FOR k := 0 TO Quantity - 1 DO
                TxBuffer[ClientIndex, 9 + k * 2] := HIGH_BYTE(InputRegisters[StartAddress + k]);
                TxBuffer[ClientIndex, 10 + k * 2] := LOW_BYTE(InputRegisters[StartAddress + k]);
            END_FOR;
            ResponseLength := 9 + ByteCount;
        ELSE
            SendException(ClientIndex, FunctionCode, 2);
            RETURN;
        END_IF;

    5: (* Write Single Coil *)
        StartAddress := BYTES_TO_UINT(RxBuffer[ClientIndex, 8], RxBuffer[ClientIndex, 9]);
        IF StartAddress < 10000 AND (RxBuffer[ClientIndex, 10] = 16#FF OR RxBuffer[ClientIndex, 10] = 0) THEN
            Coils[StartAddress] := (RxBuffer[ClientIndex, 10] = 16#FF);
            (* Echo request as response *)
            FOR k := 0 TO 11 DO
                TxBuffer[ClientIndex, k] := RxBuffer[ClientIndex, k];
            END_FOR;
            ResponseLength := 12;
        ELSE
            SendException(ClientIndex, FunctionCode, 2);
            RETURN;
        END_IF;

    6: (* Write Single Register *)
        StartAddress := BYTES_TO_UINT(RxBuffer[ClientIndex, 8], RxBuffer[ClientIndex, 9]);
        IF StartAddress < 10000 THEN
            HoldingRegisters[StartAddress] := BYTES_TO_UINT(RxBuffer[ClientIndex, 10], RxBuffer[ClientIndex, 11]);
            FOR k := 0 TO 11 DO
                TxBuffer[ClientIndex, k] := RxBuffer[ClientIndex, k];
            END_FOR;
            ResponseLength := 12;
        ELSE
            SendException(ClientIndex, FunctionCode, 2);
            RETURN;
        END_IF;

    15: (* Write Multiple Coils *)
        StartAddress := BYTES_TO_UINT(RxBuffer[ClientIndex, 8], RxBuffer[ClientIndex, 9]);
        Quantity := BYTES_TO_UINT(RxBuffer[ClientIndex, 10], RxBuffer[ClientIndex, 11]);
        ByteCount := RxBuffer[ClientIndex, 12];
        IF StartAddress + Quantity <= 10000 AND ByteCount = (Quantity + 7) / 8 THEN
            FOR k := 0 TO Quantity - 1 DO
                Coils[StartAddress + k] := (RxBuffer[ClientIndex, 13 + (k / 8)] AND (1 << (k MOD 8))) <> 0;
            END_FOR;
            TxBuffer[ClientIndex, 0] := MBAP[0];
            TxBuffer[ClientIndex, 1] := MBAP[1];
            TxBuffer[ClientIndex, 2] := MBAP[2];
            TxBuffer[ClientIndex, 3] := MBAP[3];
            TxBuffer[ClientIndex, 4] := 0;
            TxBuffer[ClientIndex, 5] := 6;
            TxBuffer[ClientIndex, 6] := MBAP[6];
            TxBuffer[ClientIndex, 7] := FunctionCode;
            TxBuffer[ClientIndex, 8] := RxBuffer[ClientIndex, 8];
            TxBuffer[ClientIndex, 9] := RxBuffer[ClientIndex, 9];
            TxBuffer[ClientIndex, 10] := RxBuffer[ClientIndex, 10];
            TxBuffer[ClientIndex, 11] := RxBuffer[ClientIndex, 11];
            ResponseLength := 12;
        ELSE
            SendException(ClientIndex, FunctionCode, 2);
            RETURN;
        END_IF;

    16: (* Write Multiple Registers *)
        StartAddress := BYTES_TO_UINT(RxBuffer[ClientIndex, 8], RxBuffer[ClientIndex, 9]);
        Quantity := BYTES_TO_UINT(RxBuffer[ClientIndex, 10], RxBuffer[ClientIndex, 11]);
        ByteCount := RxBuffer[ClientIndex, 12];
        IF StartAddress + Quantity <= 10000 AND ByteCount = Quantity * 2 THEN
            FOR k := 0 TO Quantity - 1 DO
                HoldingRegisters[StartAddress + k] := BYTES_TO_UINT(RxBuffer[ClientIndex, 13 + k * 2], RxBuffer[ClientIndex, 14 + k * 2]);
            END_FOR;
            TxBuffer[ClientIndex, 0] := MBAP[0];
            TxBuffer[ClientIndex, 1] := MBAP[1];
            TxBuffer[ClientIndex, 2] := MBAP[2];
            TxBuffer[ClientIndex, 3] := MBAP[3];
            TxBuffer[ClientIndex, 4] := 0;
            TxBuffer[ClientIndex, 5] := 6;
            TxBuffer[ClientIndex, 6] := MBAP[6];
            TxBuffer[ClientIndex, 7] := FunctionCode;
            TxBuffer[ClientIndex, 8] := RxBuffer[ClientIndex, 8];
            TxBuffer[ClientIndex, 9] := RxBuffer[ClientIndex, 9];
            TxBuffer[ClientIndex, 10] := RxBuffer[ClientIndex, 10];
            TxBuffer[ClientIndex, 11] := RxBuffer[ClientIndex, 11];
            ResponseLength := 12;
        ELSE
            SendException(ClientIndex, FunctionCode, 2);
            RETURN;
        END_IF;

    23: (* Read/Write Multiple Registers *)
        (* Read *)
        StartAddress := BYTES_TO_UINT(RxBuffer[ClientIndex, 8], RxBuffer[ClientIndex, 9]);
        Quantity := BYTES_TO_UINT(RxBuffer[ClientIndex, 10], RxBuffer[ClientIndex, 11]);
        (* Write *)
        StartAddress := BYTES_TO_UINT(RxBuffer[ClientIndex, 12], RxBuffer[ClientIndex, 13]);
        Quantity := BYTES_TO_UINT(RxBuffer[ClientIndex, 14], RxBuffer[ClientIndex, 15]);
        ByteCount := RxBuffer[ClientIndex, 16];
        IF StartAddress + Quantity <= 10000 AND ByteCount = Quantity * 2 THEN
            (* Perform write *)
            FOR k := 0 TO Quantity - 1 DO
                HoldingRegisters[StartAddress + k] := BYTES_TO_UINT(RxBuffer[ClientIndex, 17 + k * 2], RxBuffer[ClientIndex, 18 + k * 2]);
            END_FOR;
            (* Prepare read response *)
            TxBuffer[ClientIndex, 0] := MBAP[0];
            TxBuffer[ClientIndex, 1] := MBAP[1];
            TxBuffer[ClientIndex, 2] := MBAP[2];
            TxBuffer[ClientIndex, 3] := MBAP[3];
            ByteCount := Quantity * 2;
            TxBuffer[ClientIndex, 4] := HIGH_BYTE(3 + ByteCount);
            TxBuffer[ClientIndex, 5] := LOW_BYTE(3 + ByteCount);
            TxBuffer[ClientIndex, 6] := MBAP[6];
            TxBuffer[ClientIndex, 7] := FunctionCode;
            TxBuffer[ClientIndex, 8] := ByteCount;
            FOR k := 0 TO Quantity - 1 DO
                TxBuffer[ClientIndex, 9 + k * 2] := HIGH_BYTE(HoldingRegisters[StartAddress + k]);
                TxBuffer[ClientIndex, 10 + k * 2] := LOW_BYTE(HoldingRegisters[StartAddress + k]);
            END_FOR;
            ResponseLength := 9 + ByteCount;
        ELSE
            SendException(ClientIndex, FunctionCode, 2);
            RETURN;
        END_IF;

    ELSE
        SendException(ClientIndex, FunctionCode, 1); (* Illegal function *)
        RETURN;
END_CASE;

(* Send response *)
IF ResponseLength > 0 THEN
    LastError := SysSocketSend(ClientHandles[ClientIndex], TxBuffer[ClientIndex], ResponseLength, TimeoutMs);
    IF LastError <> 0 THEN
        Error := TRUE;
        ErrorID := 102; (* Send error *)
        SysSocketClose(ClientHandles[ClientIndex]);
        ClientHandles[ClientIndex] := 0;
        ClientStates[ClientIndex] := 0;
        ConnectedClients := ConnectedClients - 1;
    END_IF;
END_IF;
END_METHOD

(* Send Modbus exception *)
METHOD SendException
VAR_INPUT
    ClientIndex : UINT;
    FunctionCode : BYTE;
    ExceptionCode : BYTE;
END_VAR
TxBuffer[ClientIndex, 0] := MBAP[0];
TxBuffer[ClientIndex, 1] := MBAP[1];
TxBuffer[ClientIndex, 2] := MBAP[2];
TxBuffer[ClientIndex, 3] := MBAP[3];
TxBuffer[ClientIndex, 4] := 0;
TxBuffer[ClientIndex, 5] := 3;
TxBuffer[ClientIndex, 6] := MBAP[6];
TxBuffer[ClientIndex, 7] := FunctionCode OR 16#80;
TxBuffer[ClientIndex, 8] := ExceptionCode;
LastError := SysSocketSend(ClientHandles[ClientIndex], TxBuffer[ClientIndex], 9, TimeoutMs);
IF LastError <> 0 THEN
    Error := TRUE;
    ErrorID := 102;
    SysSocketClose(ClientHandles[ClientIndex]);
    ClientHandles[ClientIndex] := 0;
    ClientStates[ClientIndex] := 0;
    ConnectedClients := ConnectedClients - 1;
END_IF;
END_METHOD

(* Utility functions *)
METHOD BYTES_TO_UINT : UINT
VAR_INPUT
    HighByte, LowByte : BYTE;
END_VAR
BYTES_TO_UINT := (UINT_OF_BYTE(HighByte) << 8) OR UINT_OF_BYTE(LowByte);
END_METHOD

METHOD HIGH_BYTE : BYTE
VAR_INPUT
    Value : UINT;
END_VAR
HIGH_BYTE := BYTE_OF_UINT(Value >> 8);
END_METHOD

METHOD LOW_BYTE : BYTE
VAR_INPUT
    Value : UINT;
END_VAR
LOW_BYTE := BYTE_OF_UINT(Value AND 16#FF);
END_METHOD

END_FUNCTION_BLOCK
