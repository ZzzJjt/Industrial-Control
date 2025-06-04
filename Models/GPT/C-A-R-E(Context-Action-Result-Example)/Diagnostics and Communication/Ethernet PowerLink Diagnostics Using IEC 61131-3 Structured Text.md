FUNCTION_BLOCK FB_PowerLinkDiagnostics
VAR_INPUT
    NodeID            : INT;        // ID of the target PowerLink control node
    Enable            : BOOL;       // Trigger for diagnostics retrieval
END_VAR
VAR_OUTPUT
    CommStatus        : BOOL;       // Communication status: TRUE = OK, FALSE = Error
    ErrorCode         : INT;        // Latest error code reported by the node
    NodeHealth        : INT;        // Health indicator (0 = OK, >0 = warning/error)
    NewDataAvailable  : BOOL;       // Indicates fresh diagnostic data available
END_VAR
VAR
    PrevEnable        : BOOL;
    InternalBuffer    : ARRAY [0..9] OF BYTE;  // Raw diagnostic data
    CommSuccess       : BOOL;                  // Flag for communication success
    ReadOK            : BOOL;                  // Internal status
END_VAR
