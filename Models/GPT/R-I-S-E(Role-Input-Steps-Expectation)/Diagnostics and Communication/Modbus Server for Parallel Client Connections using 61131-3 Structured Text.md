FUNCTION_BLOCK FB_ModbusTCP_Server
VAR_INPUT
    Enable         : BOOL; // Enable server operation
END_VAR

VAR_OUTPUT
    ClientStatus   : ARRAY[1..10] OF BOOL;  // TRUE if client connected
    ErrorCode      : ARRAY[1..10] OF INT;   // Error code per client
END_VAR

VAR
    Socket         : ARRAY[1..10] OF INT;           // TCP socket handles
    ClientIP       : ARRAY[1..10] OF STRING[15];    // IPs of connected clients
    CoilBuffer     : ARRAY[0..1023] OF BOOL;        // Coil values
    RxBuffer       : ARRAY[1..10, 0..255] OF BYTE;  // Receive buffers
    TxBuffer       : ARRAY[1..10, 0..255] OF BYTE;  // Transmit buffers
    RequestLen     : ARRAY[1..10] OF INT;           // Length of each request
    i              : INT;                           // Loop index
    MB_UnitID      : BYTE := 1;                     // Modbus slave address
END_VAR

FOR i := 1 TO 10 DO
    IF Enable THEN
        // 1. Check and accept TCP connection
        AcceptClient(i := i, socket => Socket[i], ipAddr => ClientIP[i], connected => ClientStatus[i]);

        // 2. If connected and data received
        IF ClientStatus[i] AND ReceiveData(Socket[i], RxBuffer[i], RequestLen[i]) THEN
            CASE RxBuffer[i][7] OF // Function code at MBAP + 7
                16#01: ReadCoils(i);
                16#03: ReadHoldingRegisters(i);
                16#05: WriteSingleCoil(i);
                // Add other handlers as needed...
                ELSE
                    SendException(i, RxBuffer[i][7], 16#01); // Illegal function
            END_CASE
        END_IF
    END_IF
END_FOR

METHOD PRIVATE ReadCoils
VAR_INPUT
    ClientID : INT;
END_VAR

VAR
    StartAddr : INT;
    Quantity  : INT;
    ByteCount : INT;
    CoilByte  : BYTE;
    BitIndex  : INT;
    CoilIndex : INT;
    CoilBit   : BOOL;
    j, k      : INT;
END_VAR

// 1. Parse starting address and quantity (Big Endian)
StartAddr := SHL(TO_INT(RxBuffer[ClientID][8]), 8) + TO_INT(RxBuffer[ClientID][9]);
Quantity  := SHL(TO_INT(RxBuffer[ClientID][10]), 8) + TO_INT(RxBuffer[ClientID][11]);

IF (Quantity < 1) OR (Quantity > 2000) THEN
    SendException(ClientID, 16#01, 16#03); // Illegal data value
    RETURN;
END_IF

// 2. Calculate response length
ByteCount := (Quantity + 7) / 8; // Round up to nearest byte
TxBuffer[ClientID][0..5] := RxBuffer[ClientID][0..5]; // Copy MBAP
TxBuffer[ClientID][4] := 0;
TxBuffer[ClientID][5] := BYTE#3 + BYTE(ByteCount);   // Length = 3 + ByteCount
TxBuffer[ClientID][6] := MB_UnitID;                  // Unit ID
TxBuffer[ClientID][7] := 16#01;                      // Function Code
TxBuffer[ClientID][8] := BYTE(ByteCount);            // Byte Count

// 3. Pack coil data
j := 0;
FOR k := 0 TO ByteCount - 1 DO
    CoilByte := 0;
    FOR BitIndex := 0 TO 7 DO
        CoilIndex := StartAddr + j;
        IF (CoilIndex <= 1023) THEN
            CoilBit := CoilBuffer[CoilIndex];
            CoilByte := CoilByte OR SHL(TO_BYTE(CoilBit), BitIndex);
        END_IF
        j := j + 1;
        IF j >= Quantity THEN EXIT; END_IF;
    END_FOR
    TxBuffer[ClientID][9 + k] := CoilByte;
END_FOR

// 4. Send response
SendData(Socket[ClientID], TxBuffer[ClientID], 9 + ByteCount);
