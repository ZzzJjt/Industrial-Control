CONST
    MAX_MB_CLIENTS := 10;
    MB_HOLD_REGISTER_SIZE := 128;
    MB_COIL_SIZE := 128;
END_CONST

TYPE T_MB_SERVER_STATE :
(
    IDLE = 0,
    CONNECTING = 1,
    ACTIVE = 2,
    ERROR = 3
);
END_TYPE

TYPE T_MB_REQUEST :
STRUCT
    clientID: INT;             // Client index
    buffer: ARRAY [0..255] OF BYTE; // Raw request buffer
    length: INT;               // Length of received data
END_STRUCT
END_TYPE

FUNCTION_BLOCK FB_ModbusTCPServer
VAR_INPUT
    PORT: UINT := 502;                         // Default Modbus port
    ENABLE: BOOL := TRUE;                      // Enable server operation
END_VAR

VAR_OUTPUT
    CLIENT_CONNECTED: ARRAY [0..MAX_MB_CLIENTS - 1] OF BOOL := [FALSE]; // Connection status
    REQUEST_COUNT: DINT := 0;                  // Total number of processed requests
    LAST_ERROR: STRING(80) := '';              // Last error message
END_VAR

VAR
    serverSocket: SOCKET;                      // Server listening socket
    clientSockets: ARRAY [0..MAX_MB_CLIENTS - 1] OF SOCKET; // Client sockets
    clientStates: ARRAY [0..MAX_MB_CLIENTS - 1] OF T_MB_SERVER_STATE := [IDLE, IDLE, IDLE, IDLE, IDLE, IDLE, IDLE, IDLE, IDLE, IDLE];
    requestQueue: FIFO(T_MB_REQUEST);          // Request queue for processing
    tempBuffer: ARRAY [0..255] OF BYTE;        // Temporary read buffer
    tempLength: INT := 0;                      // Length of current read
    clientIndex: INT := 0;                     // Loop variable for clients
    mbRequest: T_MB_REQUEST;                   // Current request
    coils: ARRAY [0..MB_COIL_SIZE - 1] OF BOOL := [FALSE]; // Internal coil memory
    holdingRegisters: ARRAY [0..MB_HOLD_REGISTER_SIZE - 1] OF UINT := [0]; // Holding registers
END_VAR

// Initialize server if enabled
IF ENABLE AND NOT serverSocket.open THEN
    serverSocket(OPEN := TRUE, PORT := PORT);
END_IF;

// Accept new client connections
FOR clientIndex := 0 TO MAX_MB_CLIENTS - 1 DO
    CASE clientStates[clientIndex] OF
        IDLE:
            IF serverSocket.connected THEN
                clientSockets[clientIndex](ACCEPT := TRUE, IP_ADDR := ADR(tempIP), PORT := ADR(tempPort));
                IF clientSockets[clientIndex].connected THEN
                    clientStates[clientIndex] := ACTIVE;
                    CLIENT_CONNECTED[clientIndex] := TRUE;
                END_IF;
            END_IF;

        ACTIVE:
            clientSockets[clientIndex](RECV := TRUE, DATA := ADR(tempBuffer), SIZE := SIZEOF(tempBuffer), R_LEN := ADR(tempLength));
            IF tempLength > 0 THEN
                mbRequest.clientID := clientIndex;
                mbRequest.length := tempLength;
                MEMCPY(DEST := ADR(mbRequest.buffer), SRC := ADR(tempBuffer), SIZE := tempLength);
                requestQueue.PUT(mbRequest);
            END_IF;

            IF NOT clientSockets[clientIndex].connected THEN
                clientStates[clientIndex] := IDLE;
                CLIENT_CONNECTED[clientIndex] := FALSE;
            END_IF;

        ERROR:
            ; // Handle error state as needed
    END_CASE;
END_FOR;

// Process queued requests
WHILE requestQueue.GET(AVR := ADR(mbRequest)) DO
    ParseAndHandleModbusRequest(ADR(mbRequest));
    REQUEST_COUNT := REQUEST_COUNT + 1;
END_WHILE;

METHOD PRIVATE ParseAndHandleModbusRequest
VAR_INPUT
    req: REFERENCE TO T_MB_REQUEST;
END_VAR

VAR
    clientID: INT := req^.clientID;
    buffer: REFERENCE TO ARRAY [0..255] OF BYTE := req^.buffer;
    length: INT := req^.length;
    functionCode: BYTE := buffer[7];
    response: ARRAY [0..255] OF BYTE;
    responseLen: INT := 0;
END_VAR

// Basic sanity check
IF length < 8 THEN
    RETURN; // Invalid packet
END_IF;

CASE functionCode OF
    1: // Read Coils (0x01)
        ReadCoils(buffer, length, ADR(response), ADR(responseLen));

    2: // Read Discrete Inputs (0x02)
        ReadDiscreteInputs(buffer, length, ADR(response), ADR(responseLen));

    3: // Read Holding Registers (0x03)
        ReadHoldingRegisters(buffer, length, ADR(response), ADR(responseLen));

    4: // Read Input Registers (0x04)
        ReadInputRegisters(buffer, length, ADR(response), ADR(responseLen));

    5: // Write Single Coil (0x05)
        WriteSingleCoil(buffer, length, ADR(response), ADR(responseLen));

    6: // Write Single Register (0x06)
        WriteSingleRegister(buffer, length, ADR(response), ADR(responseLen));

    15: // Write Multiple Coils (0x0F)
        WriteMultipleCoils(buffer, length, ADR(response), ADR(responseLen));

    16: // Write Multiple Registers (0x10)
        WriteMultipleRegisters(buffer, length, ADR(response), ADR(responseLen));

    23: // Read/Write Multiple Registers (0x17)
        ReadWriteMultipleRegisters(buffer, length, ADR(response), ADR(responseLen));

    ELSE
        // Unsupported function code
        GenerateExceptionResponse(buffer, functionCode, 1, ADR(response), ADR(responseLen)); // Illegal Function
END_CASE;

// Send response back to client
IF responseLen > 0 THEN
    clientSockets[clientID](SEND := TRUE, DATA := ADR(response), SIZE := responseLen);
END_IF;

METHOD PRIVATE ReadCoils
VAR_INPUT
    reqBuffer: REFERENCE TO ARRAY [0..255] OF BYTE;
    reqLength: INT;
END_VAR
VAR_OUTPUT
    resBuffer: REFERENCE TO ARRAY [0..255] OF BYTE;
    resLength: REFERENCE TO INT;
END_VAR

VAR
    transactionID: UINT := (reqBuffer[0] SHL 8) OR reqBuffer[1];
    unitID: BYTE := reqBuffer[6];
    startAddr: UINT := (reqBuffer[8] SHL 8) OR reqBuffer[9];
    quantity: UINT := (reqBuffer[10] SHL 8) OR reqBuffer[11];
    byteCount: BYTE := BYTE_CEIL(quantity / 8);
    i: UINT;
    bitOffset: UINT;
    byteIndex: UINT;
    coilByte: BYTE;
END_VAR

// Validate coil range
IF (startAddr + quantity) > MB_COIL_SIZE THEN
    GenerateExceptionResponse(reqBuffer, 1, 2, resBuffer, resLength); // Illegal Data Address
    RETURN;
END_IF;

// Build response header
resBuffer[0] := reqBuffer[0]; // Transaction ID
resBuffer[1] := reqBuffer[1];
resBuffer[2] := 0; // Protocol ID Lo
resBuffer[3] := 0; // Protocol ID Hi
resBuffer[4] := 0; // Byte count Hi
resBuffer[5] := byteCount + 3; // Byte count Lo
resBuffer[6] := unitID;
resBuffer[7] := 1; // Function code
resBuffer[8] := byteCount;

// Pack coils into bytes
FOR i := 0 TO (byteCount - 1) DO
    coilByte := 0;
    FOR bitOffset := 0 TO 7 DO
        byteIndex := i * 8 + bitOffset;
        IF byteIndex < quantity THEN
            IF coils[startAddr + byteIndex] THEN
                coilByte := coilByte OR (1 SHL bitOffset);
            END_IF;
        END_IF;
    END_FOR;
    resBuffer[9 + i] := coilByte;
END_FOR;

resLength^ := 10 + byteCount;

METHOD PRIVATE GenerateExceptionResponse
VAR_INPUT
    reqBuffer: REFERENCE TO ARRAY [0..255] OF BYTE;
    funcCode: BYTE;
    exceptionCode: BYTE;
END_VAR
VAR_OUTPUT
    resBuffer: REFERENCE TO ARRAY [0..255] OF BYTE;
    resLength: REFERENCE TO INT;
END_VAR

resBuffer[0] := reqBuffer[0]; // Transaction ID
resBuffer[1] := reqBuffer[1];
resBuffer[2] := 0; // Protocol ID
resBuffer[3] := 0;
resBuffer[4] := 0; // Length Hi
resBuffer[5] := 3; // Length Lo
resBuffer[6] := reqBuffer[6]; // Unit ID
resBuffer[7] := funcCode OR 16#80; // Exception flag
resBuffer[8] := exceptionCode;
resLength^ := 9;
