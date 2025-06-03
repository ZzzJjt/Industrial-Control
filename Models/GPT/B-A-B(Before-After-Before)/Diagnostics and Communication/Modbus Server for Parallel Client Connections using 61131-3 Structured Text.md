FUNCTION_BLOCK MODBUS_TCP_SERVER
VAR_INPUT
    ENABLE         : BOOL;
    LISTEN_PORT    : INT := 502;   // Modbus TCP default port
END_VAR

VAR_OUTPUT
    CLIENT_COUNT   : INT;
    ERROR          : BOOL;
    STATUS         : STRING[100];
END_VAR

VAR
    // Shared memory (coils, holding/input registers)
    COILS          : ARRAY[0..1023] OF BOOL;
    HOLD_REGS      : ARRAY[0..511] OF WORD;
    INPUT_REGS     : ARRAY[0..511] OF WORD;

    // Client connections
    CLIENT_CONNECTED : ARRAY[1..10] OF BOOL;
    CLIENT_REQ       : ARRAY[1..10] OF BYTE;     // Raw Modbus request frame
    CLIENT_RES       : ARRAY[1..10] OF BYTE;     // Raw Modbus response frame
    CLIENT_BUSY      : ARRAY[1..10] OF BOOL;
    CLIENT_TIMEOUT   : ARRAY[1..10] OF TIME;
    i                : INT;

    // Temporary decode variables
    FCODE            : BYTE;
    ADDR             : WORD;
    LEN              : WORD;
    DATA             : ARRAY[0..250] OF BYTE;

    // Internal flags
    CurrentClient    : INT;
    FrameLen         : INT;
END_VAR
