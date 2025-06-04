FUNCTION_BLOCK FB_ModbusTCPServer
VAR_INPUT
    Enable       : BOOL;
    TCP_Port     : INT := 502;  // Standard Modbus TCP port
END_VAR
VAR_OUTPUT
    ConnectedClients : INT;     // Number of active connections
    ServerStatus     : STRING[50];  // Status log for diagnostics
END_VAR
VAR
    SocketPool       : ARRAY[1..10] OF INT;
    ClientActive     : ARRAY[1..10] OF BOOL;
    ReceiveBuffer    : ARRAY[1..10] OF ARRAY[0..255] OF BYTE;
    SendBuffer       : ARRAY[1..10] OF ARRAY[0..255] OF BYTE;
    HoldingRegisters : ARRAY[0..511] OF WORD;
    InputRegisters   : ARRAY[0..511] OF WORD;
    Coils            : ARRAY[0..511] OF BOOL;
    DiscreteInputs   : ARRAY[0..511] OF BOOL;
END_VAR

IF Enable THEN
    FOR i := 1 TO 10 DO
        // Step 1: Check for new or existing connection
        IF NOT ClientActive[i] THEN
            SocketPool[i] := TCP_Listen(Port := TCP_Port, ClientID := i);
            IF SocketPool[i] <> -1 THEN
                ClientActive[i] := TRUE;
                ServerStatus := CONCAT('Client Connected on ID: ', INT_TO_STRING(i));
            END_IF
        END_IF

        // Step 2: If connected, handle requests
        IF ClientActive[i] THEN
            IF TCP_Receive(SocketPool[i], ReceiveBuffer[i], Size := 256) THEN
                CASE ReceiveBuffer[i][7] OF  // Function code index
                    16#01: Handle_ReadCoils(i);
                    16#02: Handle_ReadDiscreteInputs(i);
                    16#03: Handle_ReadHoldingRegisters(i);
                    16#04: Handle_ReadInputRegisters(i);
                    16#05: Handle_WriteSingleCoil(i);
                    16#06: Handle_WriteSingleRegister(i);
                    16#0F: Handle_WriteMultipleCoils(i);
                    16#10: Handle_WriteMultipleRegisters(i);
                    16#17: Handle_ReadWriteMultipleRegisters(i);
                    ELSE
                        SendException(i, Code := 16#01); // Illegal function
                END_CASE
            ELSIF TCP_Closed(SocketPool[i]) THEN
                ClientActive[i] := FALSE;
                SocketPool[i] := -1;
                ServerStatus := CONCAT('Client Disconnected: ', INT_TO_STRING(i));
            END_IF
        END_IF
    END_FOR
END_IF

METHOD Handle_ReadCoils
VAR_INPUT ClientID : INT;
VAR
    StartAddr : INT;
    Quantity  : INT;
    ByteCount : INT;
    i         : INT;
BEGIN
    StartAddr := WORD_TO_INT(SHL(ReceiveBuffer[ClientID][8], 8)) + ReceiveBuffer[ClientID][9];
    Quantity  := WORD_TO_INT(SHL(ReceiveBuffer[ClientID][10], 8)) + ReceiveBuffer[ClientID][11];
    ByteCount := (Quantity + 7) / 8;

    SendBuffer[ClientID][0..6] := ReceiveBuffer[ClientID][0..6]; // Echo header
    SendBuffer[ClientID][7] := 16#01; // Function Code
    SendBuffer[ClientID][8] := INT_TO_BYTE(ByteCount);

    FOR i := 0 TO Quantity - 1 DO
        IF Coils[StartAddr + i] THEN
            SendBuffer[ClientID][9 + (i / 8)] := SET_BIT(SendBuffer[ClientID][9 + (i / 8)], i MOD 8);
        END_IF
    END_FOR

    TCP_Send(SocketPool[ClientID], SendBuffer[ClientID], Size := 9 + ByteCount);
END_METHOD
