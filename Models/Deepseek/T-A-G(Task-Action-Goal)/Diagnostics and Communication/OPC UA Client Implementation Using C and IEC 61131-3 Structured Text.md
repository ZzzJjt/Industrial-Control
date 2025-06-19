FUNCTION_BLOCK OPC_UA_CLIENT
VAR_INPUT
    Execute : BOOL;          // Triggers the connection
    ServerUrl : STRING[255]; // Specifies the OPC UA server URL
    Timeout : TIME;           // Sets the connection timeout
END_VAR

VAR_OUTPUT
    Done : BOOL;             // Set when the operation completes successfully
    Busy : BOOL;             // Indicates ongoing operation
    Error : BOOL;            // Flags a failure
    ErrorID : DWORD;         // Provides detailed error information
END_VAR

VAR
    lastExecute : BOOL := FALSE; // Last state of Execute input
    opcuaClientHandle : REFERENCE; // Handle to the OPC UA client
    connectAttempted : BOOL := FALSE; // Flag to indicate if connection has been attempted
    connectSuccess : BOOL := FALSE; // Flag to indicate if connection was successful
    connectError : DWORD := 0; // Error code for connection attempt
END_VAR

METHOD ConnectToServer : BOOL
VAR_INPUT
    url : STRING[255];
END_VAR
VAR
    success : BOOL := FALSE;
BEGIN
    IF opcua_connect(url, ADR(opcuaClientHandle)) THEN
        success := TRUE;
    ELSE
        connectError := 1; // Example error code
    END_IF;
    RETURN success;
END_METHOD

METHOD DisconnectFromServer : BOOL
BEGIN
    opcua_disconnect(opcuaClientHandle);
    RETURN TRUE;
END_METHOD

METHOD Execute : BOOL
BEGIN
    IF R_TRIG(lastExecute, Execute).Q THEN
        IF NOT connectAttempted THEN
            Busy := TRUE;
            Done := FALSE;
            Error := FALSE;
            connectSuccess := ConnectToServer(ServerUrl);
            connectAttempted := TRUE;
            IF connectSuccess THEN
                Done := TRUE;
                Busy := FALSE;
            ELSE
                Error := TRUE;
                ErrorID := connectError;
                Busy := FALSE;
            END_IF;
        END_IF;
    ELSIF NOT Execute THEN
        lastExecute := FALSE;
        connectAttempted := FALSE;
        connectSuccess := FALSE;
        connectError := 0;
        Done := FALSE;
        Busy := FALSE;
        Error := FALSE;
        ErrorID := 0;
    END_IF;

    lastExecute := Execute;
    RETURN TRUE;
END_METHOD

END_FUNCTION_BLOCK



