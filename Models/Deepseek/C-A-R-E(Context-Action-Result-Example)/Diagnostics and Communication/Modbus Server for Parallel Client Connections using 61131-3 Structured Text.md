FUNCTION_BLOCK Modbus_TCP_Server
VAR_INPUT
    Enable : BOOL; // Enable the server
END_VAR

VAR_OUTPUT
    ServerStatus : STRING[50]; // Status of the server
END_VAR

VAR
    Clients : ARRAY[1..10] OF CLIENT_CONNECTION;
    ClientSockets : ARRAY[1..10] OF INT; // Socket handles for each client
    NumClients : INT := 0; // Number of connected clients
    ListenSocket : INT; // Socket handle for listening
    Buffer : ARRAY[1..256] OF BYTE; // Buffer for receiving/sending data
    ReceiveLength : INT; // Length of received data
    SendLength : INT; // Length of data to send
    Timer : TON; // Timer for periodic checks
END_VAR

TYPE CLIENT_CONNECTION :
STRUCT
    Connected : BOOL;
    Address : STRING[20];
    Port : INT;
END_STRUCT
END_TYPE

// Method to initialize the server
METHOD InitializeServer : BOOL
VAR
    Result : INT;
END_VAR
    ListenSocket := SOCKET_CREATE(AF_INET, SOCK_STREAM, IPPROTO_TCP);
    IF ListenSocket = -1 THEN
        ServerStatus := 'Failed to create socket';
        RETURN FALSE;
    END_IF;

    Result := SOCKET_BIND(ListenSocket, '0.0.0.0', 502); // Bind to port 502
    IF Result = -1 THEN
        ServerStatus := 'Failed to bind socket';
        SOCKET_CLOSE(ListenSocket);
        RETURN FALSE;
    END_IF;

    Result := SOCKET_LISTEN(ListenSocket, 10); // Listen with backlog of 10
    IF Result = -1 THEN
        ServerStatus := 'Failed to listen on socket';
        SOCKET_CLOSE(ListenSocket);
        RETURN FALSE;
    END_IF;

    ServerStatus := 'Server initialized';
    RETURN TRUE;
END_METHOD

// Method to accept new clients
METHOD AcceptClient : BOOL
VAR
    NewSocket : INT;
    ClientAddr : STRING[20];
    ClientPort : INT;
    i : INT;
END_VAR
    IF NumClients >= 10 THEN
        RETURN FALSE; // Maximum number of clients reached
    END_IF;

    NewSocket := SOCKET_ACCEPT(ListenSocket, ClientAddr, ClientPort);
    IF NewSocket = -1 THEN
        RETURN FALSE; // No new client
    END_IF;

    FOR i := 1 TO 10 DO
        IF NOT Clients[i].Connected THEN
            Clients[i].Connected := TRUE;
            Clients[i].Address := ClientAddr;
            Clients[i].Port := ClientPort;
            ClientSockets[i] := NewSocket;
            NumClients := NumClients + 1;
            RETURN TRUE;
        END_IF;
    END_FOR;

    SOCKET_CLOSE(NewSocket); // Close the socket if no slot available
    RETURN FALSE;
END_METHOD

// Method to handle client requests
METHOD HandleClientRequest : BOOL
VAR_INPUT
    ClientIndex : INT;
END_VAR
VAR
    FunctionCode : BYTE;
    StartingAddress : UINT;
    Quantity : UINT;
    ResponseBuffer : ARRAY[1..256] OF BYTE;
    ResponseLength : INT;
    Result : INT;
END_VAR
    IF NOT Clients[ClientIndex].Connected THEN
        RETURN FALSE;
    END_IF;

    ReceiveLength := SOCKET_RECEIVE(ClientSockets[ClientIndex], Buffer, SIZEOF(Buffer));
    IF ReceiveLength <= 0 THEN
        DisconnectClient(ClientIndex);
        RETURN FALSE;
    END_IF;

    // Parse Modbus request
    FunctionCode := Buffer[8];
    StartingAddress := WORD_TO_UINT(BUFFER_TO_WORD(Buffer[9], Buffer[10]));
    Quantity := WORD_TO_UINT(BUFFER_TO_WORD(Buffer[11], Buffer[12]));

    CASE FunctionCode OF
        16#01: // Read Coils
            ResponseLength := BuildReadCoilsResponse(ResponseBuffer, StartingAddress, Quantity);
        16#02: // Read Discrete Inputs
            ResponseLength := BuildReadDiscreteInputsResponse(ResponseBuffer, StartingAddress, Quantity);
        16#03: // Read Holding Registers
            ResponseLength := BuildReadHoldingRegistersResponse(ResponseBuffer, StartingAddress, Quantity);
        16#04: // Read Input Registers
            ResponseLength := BuildReadInputRegistersResponse(ResponseBuffer, StartingAddress, Quantity);
        16#05: // Write Single Coil
            ResponseLength := BuildWriteSingleCoilResponse(ResponseBuffer, StartingAddress, Buffer[13]);
        16#06: // Write Single Register
            ResponseLength := BuildWriteSingleRegisterResponse(ResponseBuffer, StartingAddress, BUFFER_TO_WORD(Buffer[13], Buffer[14]));
        16#0F: // Write Multiple Coils
            ResponseLength := BuildWriteMultipleCoilsResponse(ResponseBuffer, StartingAddress, Quantity, Buffer[15..ReceiveLength]);
        16#10: // Write Multiple Registers
            ResponseLength := BuildWriteMultipleRegistersResponse(ResponseBuffer, StartingAddress, Quantity, Buffer[15..ReceiveLength]);
        16#17: // Read/Write Multiple Registers
            ResponseLength := BuildReadWriteMultipleRegistersResponse(ResponseBuffer, StartingAddress, Quantity, Buffer[15..ReceiveLength]);
        ELSE
            ResponseLength := BuildExceptionResponse(ResponseBuffer, FunctionCode, 1); // Illegal Function
    END_CASE;

    IF ResponseLength > 0 THEN
        SendLength := SOCKET_SEND(ClientSockets[ClientIndex], ResponseBuffer, ResponseLength);
        IF SendLength = -1 THEN
            DisconnectClient(ClientIndex);
            RETURN FALSE;
        END_IF;
    END_IF;

    RETURN TRUE;
END_METHOD

// Method to disconnect a client
METHOD DisconnectClient : BOOL
VAR_INPUT
    ClientIndex : INT;
END_VAR
    IF Clients[ClientIndex].Connected THEN
        SOCKET_CLOSE(ClientSockets[ClientIndex]);
        Clients[ClientIndex].Connected := FALSE;
        NumClients := NumClients - 1;
        RETURN TRUE;
    END_IF;
    RETURN FALSE;
END_METHOD

// Helper methods to build responses for different function codes
METHOD BuildReadCoilsResponse : INT
VAR_INPUT
    ResponseBuffer : REFERENCE TO ARRAY[1..256] OF BYTE;
    StartingAddress : UINT;
    Quantity : UINT;
END_VAR
VAR
    ByteCount : BYTE;
    BitIndex : INT;
    DataIndex : INT;
    CoilValue : BOOL;
BEGIN
    ByteCount := (Quantity + 7) DIV 8;
    ResponseBuffer[1] := 1; // Function Code
    ResponseBuffer[2] := ByteCount;
    DataIndex := 3;

    FOR BitIndex := 0 TO Quantity - 1 DO
        CoilValue := READ_COIL(StartingAddress + BitIndex);
        IF CoilValue THEN
            SET_BIT(ResponseBuffer[(BitIndex DIV 8) + 3], BitIndex MOD 8);
        ELSE
            CLEAR_BIT(ResponseBuffer[(BitIndex DIV 8) + 3], BitIndex MOD 8);
        END_IF;
    END_FOR;

    BuildReadCoilsResponse := ByteCount + 2;
END_METHOD

METHOD BuildReadDiscreteInputsResponse : INT
VAR_INPUT
    ResponseBuffer : REFERENCE TO ARRAY[1..256] OF BYTE;
    StartingAddress : UINT;
    Quantity : UINT;
END_VAR
VAR
    ByteCount : BYTE;
    BitIndex : INT;
    DataIndex : INT;
    InputValue : BOOL;
BEGIN
    ByteCount := (Quantity + 7) DIV 8;
    ResponseBuffer[1] := 2; // Function Code
    ResponseBuffer[2] := ByteCount;
    DataIndex := 3;

    FOR BitIndex := 0 TO Quantity - 1 DO
        InputValue := READ_DISCRETE_INPUT(StartingAddress + BitIndex);
        IF InputValue THEN
            SET_BIT(ResponseBuffer[(BitIndex DIV 8) + 3], BitIndex MOD 8);
        ELSE
            CLEAR_BIT(ResponseBuffer[(BitIndex DIV 8) + 3], BitIndex MOD 8);
        END_IF;
    END_FOR;

    BuildReadDiscreteInputsResponse := ByteCount + 2;
END_METHOD

METHOD BuildReadHoldingRegistersResponse : INT
VAR_INPUT
    ResponseBuffer : REFERENCE TO ARRAY[1..256] OF BYTE;
    StartingAddress : UINT;
    Quantity : UINT;
END_VAR
VAR
    ByteCount : BYTE;
    RegIndex : INT;
    RegisterValue : UINT;
BEGIN
    ByteCount := Quantity * 2;
    ResponseBuffer[1] := 3; // Function Code
    ResponseBuffer[2] := ByteCount;
    DataIndex := 3;

    FOR RegIndex := 0 TO Quantity - 1 DO
        RegisterValue := READ_HOLDING_REGISTER(StartingAddress + RegIndex);
        ResponseBuffer[DataIndex] := HIGH_BYTE(RegisterValue);
        ResponseBuffer[DataIndex + 1] := LOW_BYTE(RegisterValue);
        DataIndex := DataIndex + 2;
    END_FOR;

    BuildReadHoldingRegistersResponse := ByteCount + 2;
END_METHOD

METHOD BuildReadInputRegistersResponse : INT
VAR_INPUT
    ResponseBuffer : REFERENCE TO ARRAY[1..256] OF BYTE;
    StartingAddress : UINT;
    Quantity : UINT;
END_VAR
VAR
    ByteCount : BYTE;
    RegIndex : INT;
    RegisterValue : UINT;
BEGIN
    ByteCount := Quantity * 2;
    ResponseBuffer[1] := 4; // Function Code
    ResponseBuffer[2] := ByteCount;
    DataIndex := 3;

    FOR RegIndex := 0 TO Quantity - 1 DO
        RegisterValue := READ_INPUT_REGISTER(StartingAddress + RegIndex);
        ResponseBuffer[DataIndex] := HIGH_BYTE(RegisterValue);
        ResponseBuffer[DataIndex + 1] := LOW_BYTE(RegisterValue);
        DataIndex := DataIndex + 2;
    END_FOR;

    BuildReadInputRegistersResponse := ByteCount + 2;
END_METHOD

METHOD BuildWriteSingleCoilResponse : INT
VAR_INPUT
    ResponseBuffer : REFERENCE TO ARRAY[1..256] OF BYTE;
    StartingAddress : UINT;
    Value : BYTE;
END_VAR
VAR
    CoilValue : BOOL;
BEGIN
    CoilValue := Value <> 0;
    WRITE_COIL(StartingAddress, CoilValue);

    ResponseBuffer[1] := 5; // Function Code
    ResponseBuffer[2] := HIGH_BYTE(StartingAddress);
    ResponseBuffer[3] := LOW_BYTE(StartingAddress);
    ResponseBuffer[4] := HIGH_BYTE(IF CoilValue THEN 0xFF00 ELSE 0x0000 END_INT);
    ResponseBuffer[5] := LOW_BYTE(IF CoilValue THEN 0xFF00 ELSE 0x0000 END_INT);

    BuildWriteSingleCoilResponse := 8;
END_METHOD

METHOD BuildWriteSingleRegisterResponse : INT
VAR_INPUT
    ResponseBuffer : REFERENCE TO ARRAY[1..256] OF BYTE;
    StartingAddress : UINT;
    Value : UINT;
END_VAR
BEGIN
    WRITE_HOLDING_REGISTER(StartingAddress, Value);

    ResponseBuffer[1] := 6; // Function Code
    ResponseBuffer[2] := HIGH_BYTE(StartingAddress);
    ResponseBuffer[3] := LOW_BYTE(StartingAddress);
    ResponseBuffer[4] := HIGH_BYTE(Value);
    ResponseBuffer[5] := LOW_BYTE(Value);

    BuildWriteSingleRegisterResponse := 6;
END_METHOD

METHOD BuildWriteMultipleCoilsResponse : INT
VAR_INPUT
    ResponseBuffer : REFERENCE TO ARRAY[1..256] OF BYTE;
    StartingAddress : UINT;
    Quantity : UINT;
    Data : ARRAY[1..256] OF BYTE;
END_VAR
VAR
    ByteCount : BYTE;
    BitIndex : INT;
    CoilValue : BOOL;
BEGIN
    ByteCount := (Quantity + 7) DIV 8;
    FOR BitIndex := 0 TO Quantity - 1 DO
        CoilValue := GET_BIT(Data[(BitIndex DIV 8) + 1], BitIndex MOD 8);
        WRITE_COIL(StartingAddress + BitIndex, CoilValue);
    END_FOR;

    ResponseBuffer[1] := 15; // Function Code
    ResponseBuffer[2] := HIGH_BYTE(StartingAddress);
    ResponseBuffer[3] := LOW_BYTE(StartingAddress);
    ResponseBuffer[4] := HIGH_BYTE(Quantity);
    ResponseBuffer[5] := LOW_BYTE(Quantity);

    BuildWriteMultipleCoilsResponse := 6;
END_METHOD

METHOD BuildWriteMultipleRegistersResponse : INT
VAR_INPUT
    ResponseBuffer : REFERENCE TO ARRAY[1..256] OF BYTE;
    StartingAddress : UINT;
    Quantity : UINT;
    Data : ARRAY[1..256] OF BYTE;
END_VAR
VAR
    RegIndex : INT;
    RegisterValue : UINT;
BEGIN
    FOR RegIndex := 0 TO Quantity - 1 DO
        RegisterValue := BUFFER_TO_WORD(Data[(RegIndex * 2) + 1], Data[(RegIndex * 2) + 2]);
        WRITE_HOLDING_REGISTER(StartingAddress + RegIndex, RegisterValue);
    END_FOR;

    ResponseBuffer[1] := 16; // Function Code
    ResponseBuffer[2] := HIGH_BYTE(StartingAddress);
    ResponseBuffer[3] := LOW_BYTE(StartingAddress);
    ResponseBuffer[4] := HIGH_BYTE(Quantity);
    ResponseBuffer[5] := LOW_BYTE(Quantity);

    BuildWriteMultipleRegistersResponse := 6;
END_METHOD

METHOD BuildReadWriteMultipleRegistersResponse : INT
VAR_INPUT
    ResponseBuffer : REFERENCE TO ARRAY[1..256] OF BYTE;
    StartingAddress : UINT;
    Quantity : UINT;
    Data : ARRAY[1..256] OF BYTE;
END_VAR
VAR
    ByteCount : BYTE;
    RegIndex : INT;
    RegisterValue : UINT;
BEGIN
    // Read first
    ByteCount := Quantity * 2;
    ResponseBuffer[1] := 23; // Function Code
    ResponseBuffer[2] := ByteCount;
    FOR RegIndex := 0 TO Quantity - 1 DO
        RegisterValue := READ_HOLDING_REGISTER(StartingAddress + RegIndex);
        ResponseBuffer[RegIndex * 2 + 3] := HIGH_BYTE(RegisterValue);
        ResponseBuffer[RegIndex * 2 + 4] := LOW_BYTE(RegisterValue);
    END_FOR;

    // Write then
    FOR RegIndex := 0 TO Quantity - 1 DO
        RegisterValue := BUFFER_TO_WORD(Data[(RegIndex * 2) + 1], Data[(RegIndex * 2) + 2]);
        WRITE_HOLDING_REGISTER(StartingAddress + RegIndex, RegisterValue);
    END_FOR;

    BuildReadWriteMultipleRegistersResponse := ByteCount + 2;
END_METHOD

METHOD BuildExceptionResponse : INT
VAR_INPUT
    ResponseBuffer : REFERENCE TO ARRAY[1..256] OF BYTE;
    FunctionCode : BYTE;
    ExceptionCode : BYTE;
END_VAR
BEGIN
    ResponseBuffer[1] := FunctionCode OR 128; // Set MSB to indicate exception
    ResponseBuffer[2] := ExceptionCode;

    BuildExceptionResponse := 2;
END_METHOD

// Main execution logic
IF Enable THEN
    IF InitializeServer() THEN
        Timer(IN := TRUE, PT := T#1s); // Check every second
        WHILE Timer.Q DO
            Timer(IN := FALSE); // Reset timer

            // Accept new clients
            ACCEPTCLIENT();

            // Handle existing clients
            FOR i := 1 TO 10 DO
                IF Clients[i].Connected THEN
                    HANDLECLIENTREQUEST(i);
                END_IF;
            END_FOR;
        END_WHILE;
    ELSE
        ServerStatus := 'Initialization failed';
    END_IF;
ELSE
    ServerStatus := 'Server disabled';
END_IF;



