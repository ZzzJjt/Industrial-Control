FUNCTION_BLOCK OPCUAClient
VAR_INPUT
    Execute : BOOL; // Triggers the connection process
    ServerUrl : STRING[255]; // Specifies the target OPC UA server
    Timeout : TIME; // Defines the allowable wait period for connection
END_VAR

VAR_OUTPUT
    Done : BOOL; // Indicates successful execution
    Busy : BOOL; // Signals ongoing communication
    Error : BOOL; // Flags any failure during execution
    ErrorID : DWORD; // Returns the relevant error code
END_VAR

VAR
    State : REFERENCE TO OpcUaClientState; // Reference to the C client state
    Initialized : BOOL := FALSE;
END_VAR

// External function declarations
EXTERNAL FUNCTION opcua_client_init : VOID
VAR_INPUT
    state : REFERENCE TO OpcUaClientState;
END_VAR
END_FUNCTION

EXTERNAL FUNCTION opcua_connect : DWORD
VAR_INPUT
    state : REFERENCE TO OpcUaClientState;
END_VAR
END_FUNCTION

TYPE OpcUaClientState :
STRUCT
    Execute : BOOL;
    ServerUrl : ARRAY[1..256] OF CHAR;
    Timeout : DWORD;
    Done : BOOL;
    Busy : BOOL;
    Error : BOOL;
    ErrorID : DWORD;
END_STRUCT
END_TYPE

// Main execution logic
IF NOT Initialized THEN
    State := ALLOCATE(OpcUaClientState);
    opcua_client_init(State);
    Initialized := TRUE;
END_IF;

IF Execute AND NOT Busy THEN
    Busy := TRUE;
    State^.Execute := Execute;
    MEMCOPY(State^.ServerUrl, ADR(ServerUrl), LEN(ServerUrl));
    State^.Timeout := TIMEOUT_TO_DWORD(Timeout);
    State^.Done := FALSE;
    State^.Busy := FALSE;
    State^.Error := FALSE;
    State^.ErrorID := 0;

    State^.ErrorID := opcua_connect(State);
    IF State^.ErrorID = 0 THEN
        Done := TRUE;
    ELSE
        Error := TRUE;
        ErrorID := State^.ErrorID;
    END_IF;
    Busy := FALSE;
END_IF;
