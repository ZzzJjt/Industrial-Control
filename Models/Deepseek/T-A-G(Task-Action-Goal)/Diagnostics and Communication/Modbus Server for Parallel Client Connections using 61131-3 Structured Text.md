FUNCTION_BLOCK MODBUS_TCP_SERVER
VAR_INPUT
    ENABLE : BOOL; // Enable the Modbus TCP server
END_VAR

VAR_OUTPUT
    STATUS_MESSAGE : STRING[100]; // Status message
    OVERALL_STATUS : INT;         // Overall status code (0 = OK, -1 = Error)
END_VAR

VAR
    MAX_CLIENTS : INT := 10;       // Maximum number of clients
    clientsConnected : ARRAY[1..MAX_CLIENTS] OF BOOL;
    clientTimers : ARRAY[1..MAX_CLIENTS] OF TON;
    clientTimeout : TIME := T#5s;  // Timeout for each client connection
    coilData : ARRAY[1..1000] OF BOOL; // Coil data buffer
    discreteInputData : ARRAY[1..1000] OF BOOL; // Discrete input data buffer
    holdingRegisterData : ARRAY[1..1000] OF INT; // Holding register data buffer
    inputRegisterData : ARRAY[1..1000] OF INT; // Input register data buffer
    clientRequests : ARRAY[1..MAX_CLIENTS] OF STRING[256];
    clientResponses : ARRAY[1..MAX_CLIENTS] OF STRING[256];
    currentClientIndex : INT := 1; // Current client index for cyclic processing
    processCycleComplete : BOOL := FALSE; // Flag to indicate cycle completion
    errorCode : INT := 0;          // Error code for fault handling
    errorMessage : STRING[100];   // Error message for fault handling
END_VAR

METHOD InitializeServer : BOOL
BEGIN
    // Simulate initialization of the Modbus TCP server
    // Replace with actual API calls in real application
    FOR i := 1 TO MAX_CLIENTS DO
        clientsConnected[i] := FALSE;
        clientTimers[i](IN := FALSE);
    END_FOR;
    RETURN TRUE;
END_METHOD

METHOD HandleClientRequest : BOOL
VAR_INPUT
    clientIndex : INT;
END_VAR
VAR
    request : STRING[256];
    response : STRING[256];
    functionCode : BYTE;
    startAddress : WORD;
    quantity : WORD;
    dataLength : WORD;
    byteCount : BYTE;
    coils : ARRAY[1..1000] OF BOOL;
    registers : ARRAY[1..1000] OF INT;
    success : BOOL := TRUE;
BEGIN
    request := clientRequests[clientIndex];

    IF LEN(request) < 7 THEN
        success := FALSE;
        errorCode := 1;
        errorMessage := 'Invalid request length';
        RETURN FALSE;
    END_IF;

    functionCode := BYTE_TO_UINT8(BYTE#request[8]);
    startAddress := WORD_TO_UINT16(WORD#request[9]) SHL 8 + WORD_TO_UINT16(WORD#request[10]);
    quantity := WORD_TO_UINT16(WORD#request[11]) SHL 8 + WORD_TO_UINT16(WORD#request[12]);

    CASE functionCode OF
        1: // Read Coils
            IF startAddress + quantity > SIZEOF(coilData) THEN
                success := FALSE;
                errorCode := 2;
                errorMessage := 'Illegal data address';
            ELSE
                byteCount := (quantity + 7) DIV 8;
                response := CONCAT(
                    CHR#request[1], CHR#request[2], CHR#request[3], CHR#request[4],
                    CHR#request[5], CHR#request[6], CHR#request[7], CHR(byteCount)
                );
                FOR i := 0 TO quantity - 1 DO
                    coils[i + 1] := coilData[startAddress + i];
                END_FOR;
                FOR i := 0 TO byteCount - 1 DO
                    response := CONCAT(response, CHR(PackBits(coils[(i * 8) + 1 .. MIN(quantity, (i + 1) * 8)])));
                END_FOR;
            END_IF;
        2: // Read Discrete Inputs
            IF startAddress + quantity > SIZEOF(discreteInputData) THEN
                success := FALSE;
                errorCode := 2;
                errorMessage := 'Illegal data address';
            ELSE
                byteCount := (quantity + 7) DIV 8;
                response := CONCAT(
                    CHR#request[1], CHR#request[2], CHR#request[3], CHR#request[4],
                    CHR#request[5], CHR#request[6], CHR#request[7], CHR(byteCount)
                );
                FOR i := 0 TO quantity - 1 DO
                    coils[i + 1] := discreteInputData[startAddress + i];
                END_FOR;
                FOR i := 0 TO byteCount - 1 DO
                    response := CONCAT(response, CHR(PackBits(coils[(i * 8) + 1 .. MIN(quantity, (i + 1) * 8)])));
                END_FOR;
            END_IF;
        3: // Read Holding Registers
            IF startAddress + quantity > SIZEOF(holdingRegisterData) THEN
                success := FALSE;
                errorCode := 2;
                errorMessage := 'Illegal data address';
            ELSE
                dataLength := quantity * 2;
                response := CONCAT(
                    CHR#request[1], CHR#request[2], CHR#request[3], CHR#request[4],
                    CHR#request[5], CHR#request[6], CHR#request[7], CHR(dataLength)
                );
                FOR i := 0 TO quantity - 1 DO
                    response := CONCAT(response, CHR(HOLDINGREGISTERDATA[startAddress + i] SHR 8));
                    response := CONCAT(response, CHR(HOLDINGREGISTERDATA[startAddress + i] AND 255));
                END_FOR;
            END_IF;
        4: // Read Input Registers
            IF startAddress + quantity > SIZEOF(inputRegisterData) THEN
                success := FALSE;
                errorCode := 2;
                errorMessage := 'Illegal data address';
            ELSE
                dataLength := quantity * 2;
                response := CONCAT(
                    CHR#request[1], CHR#request[2], CHR#request[3], CHR#request[4],
                    CHR#request[5], CHR#request[6], CHR#request[7], CHR(dataLength)
                );
                FOR i := 0 TO quantity - 1 DO
                    response := CONCAT(response, CHR(INPUTREGISTERDATA[startAddress + i] SHR 8));
                    response := CONCAT(response, CHR(INPUTREGISTERDATA[startAddress + i] AND 255));
                END_FOR;
            END_IF;
        5: // Write Single Coil
            IF startAddress > SIZEOF(coilData) THEN
                success := FALSE;
                errorCode := 2;
                errorMessage := 'Illegal data address';
            ELSE
                coilData[startAddress] := (UINT16_TO_BYTE(UINT16#request[9]) = 0xFF);
                response := request;
            END_IF;
        6: // Write Single Register
            IF startAddress > SIZEOF(holdingRegisterData) THEN
                success := FALSE;
                errorCode := 2;
                errorMessage := 'Illegal data address';
            ELSE
                holdingRegisterData[startAddress] := UINT16_TO_INT(INT16#request[9]) SHL 8 + UINT16_TO_INT(INT16#request[10]);
                response := request;
            END_IF;
        15: // Write Multiple Coils
            byteCount := UINT8_TO_BYTE(BYTE#request[12]);
            IF startAddress + (byteCount * 8) > SIZEOF(coilData) THEN
                success := FALSE;
                errorCode := 2;
                errorMessage := 'Illegal data address';
            ELSE
                FOR i := 0 TO byteCount - 1 DO
                    UNPACKBITS(registers[i + 1], coils[(i * 8) + 1 .. MIN(byteCount * 8, (i + 1) * 8)]);
                    FOR j := 0 TO 7 DO
                        IF (i * 8) + j + 1 <= quantity THEN
                            coilData[startAddress + (i * 8) + j] := coils[j + 1];
                        END_IF;
                    END_FOR;
                END_FOR;
                response := CONCAT(
                    CHR#request[1], CHR#request[2], CHR#request[3], CHR#request[4],
                    CHR#request[5], CHR#request[6], CHR#request[7],
                    CHR(startAddress SHR 8), CHR(startAddress AND 255),
                    CHR(quantity SHR 8), CHR(quantity AND 255)
                );
            END_IF;
        16: // Write Multiple Registers
            dataLength := UINT8_TO_BYTE(BYTE#request[12]);
            IF startAddress + (dataLength / 2) > SIZEOF(holdingRegisterData) THEN
                success := FALSE;
                errorCode := 2;
                errorMessage := 'Illegal data address';
            ELSE
                FOR i := 0 TO (dataLength / 2) - 1 DO
                    holdingRegisterData[startAddress + i] := UINT16_TO_INT(INT16#request[13 + (i * 2)]) SHL 8 + UINT16_TO_INT(INT16#request[14 + (i * 2)]);
                END_FOR;
                response := CONCAT(
                    CHR#request[1], CHR#request[2], CHR#request[3], CHR#request[4],
                    CHR#request[5], CHR#request[6], CHR#request[7],
                    CHR(startAddress SHR 8), CHR(startAddress AND 255),
                    CHR(quantity SHR 8), CHR(quantity AND 255)
                );
            END_IF;
        23: // Read/Write Multiple Registers
            readStartAddress := UINT16_TO_INT(INT16#request[9]) SHL 8 + UINT16_TO_INT(INT16#request[10]);
            readQuantity := UINT16_TO_INT(INT16#request[11]) SHL 8 + UINT16_TO_INT(INT16#request[12]);
            writeStartAddress := UINT16_TO_INT(INT16#request[13]) SHL 8 + UINT16_TO_INT(INT16#request[14]);
            writeQuantity := UINT16_TO_INT(INT16#request[15]) SHL 8 + UINT16_TO_INT(INT16#request[16]);
            byteCount := UINT8_TO_BYTE(BYTE#request[17]);
            IF readStartAddress + readQuantity > SIZEOF(inputRegisterData) OR writeStartAddress + writeQuantity > SIZEOF(holdingRegisterData) THEN
                success := FALSE;
                errorCode := 2;
                errorMessage := 'Illegal data address';
            ELSE
                FOR i := 0 TO readQuantity - 1 DO
                    registers[i + 1] := inputRegisterData[readStartAddress + i];
                END_FOR;
                FOR i := 0 TO (byteCount / 2) - 1 DO
                    holdingRegisterData[writeStartAddress + i] := UINT16_TO_INT(INT16#request[18 + (i * 2)]) SHL 8 + UINT16_TO_INT(INT16#request[19 + (i * 2)]);
                END_FOR;
                dataLength := readQuantity * 2;
                response := CONCAT(
                    CHR#request[1], CHR#request[2], CHR#request[3], CHR#request[4],
                    CHR#request[5], CHR#request[6], CHR#request[7], CHR(dataLength)
                );
                FOR i := 0 TO readQuantity - 1 DO
                    response := CONCAT(response, CHR(registers[i + 1] SHR 8));
                    response := CONCAT(response, CHR(registers[i + 1] AND 255));
                END_FOR;
            END_IF;
        ELSE
            success := FALSE;
            errorCode := 1;
            errorMessage := 'Unsupported function code';
    END_CASE;

    IF success THEN
        clientResponses[clientIndex] := response;
    ELSE
        clientResponses[clientIndex] := '';
    END_IF;

    RETURN success;
END_METHOD

METHOD PackBits : BYTE
VAR_INPUT
    bits : ARRAY[1..8] OF BOOL;
END_VAR
VAR
    packedByte : BYTE := 0;
BEGIN
    FOR i := 1 TO 8 DO
        IF bits[i] THEN
            packedByte := packedByte OR (1 SHL (8 - i));
        END_IF;
    END_FOR;
    RETURN packedByte;
END_METHOD

METHOD UnpackBits : BOOL
VAR_INPUT
    packedByte : BYTE;
END_VAR
VAR
    bits : ARRAY[1..8] OF BOOL;
BEGIN
    FOR i := 1 TO 8 DO
        bits[i] := (packedByte AND (1 SHL (8 - i))) <> 0;
    END_FOR;
    RETURN TRUE;
END_METHOD

METHOD Execute : BOOL
BEGIN
    IF NOT ENABLE THEN
        OVERALL_STATUS := 0;
        STATUS_MESSAGE := '';
        RETURN FALSE;
    END_IF;

    // Initialize server if not already initialized
    IF NOT InitializeServer() THEN
        OVERALL_STATUS := -1;
        STATUS_MESSAGE := errorMessage;
        RETURN FALSE;
    END_IF;

    // Cyclically handle each client
    IF currentClientIndex <= MAX_CLIENTS THEN
        // Start or reset the timer for the current client
        clientTimers[currentClientIndex](IN := clientsConnected[currentClientIndex], PT := clientTimeout);

        IF clientTimers[currentClientIndex].Q THEN
            // Client timed out
            clientsConnected[currentClientIndex] := FALSE;
            clientTimers[currentClientIndex](IN := FALSE);
            clientRequests[currentClientIndex] := '';
            clientResponses[currentClientIndex] := '';
            STATUS_MESSAGE := CONCAT('Client ', TO_STRING(currentClientIndex), ': Connection timeout.');
            OVERALL_STATUS := -1;
        ELSIF clientsConnected[currentClientIndex] AND clientRequests[currentClientIndex] <> '' THEN
            // Handle client request
            IF NOT HandleClientRequest(currentClientIndex) THEN
                STATUS_MESSAGE := CONCAT('Client ', TO_STRING(currentClientIndex), ': ', errorMessage);
                OVERALL_STATUS := -1;
            ELSE
                STATUS_MESSAGE := CONCAT('Client ', TO_STRING(currentClientIndex), ': Request processed successfully.');
                OVERALL_STATUS := 0;
            END_IF;
        END_IF;

        currentClientIndex := currentClientIndex + 1;
    ELSE
        currentClientIndex := 1; // Reset to start of cycle
        processCycleComplete := TRUE;
    END_IF;

    IF processCycleComplete THEN
        processCycleComplete := FALSE;
    END_IF;

    // Delay to prevent busy-waiting
    SLEEP(T#100ms);

    RETURN TRUE;
END_METHOD

END_FUNCTION_BLOCK



