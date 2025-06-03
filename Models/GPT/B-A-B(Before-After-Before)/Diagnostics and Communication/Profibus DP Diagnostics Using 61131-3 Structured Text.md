FUNCTION_BLOCK PROFIBUS_DIAG_READER
VAR_INPUT
    Execute        : BOOL;             // Rising edge triggers diagnostic read
    SlaveAddress   : BYTE;             // Profibus DP slave address (0–125)
END_VAR

VAR_OUTPUT
    Done           : BOOL;             // TRUE when read is complete
    Busy           : BOOL;             // TRUE while operation is in progress
    Error          : BOOL;             // TRUE if error occurred
    ErrorID        : WORD;             // 0 = OK, others = error code

    // Parsed diagnostic data
    StationStatus1 : BYTE;             // Bit field: mode, error, diagnostics active
    StationStatus2 : BYTE;
    StationStatus3 : BYTE;
    MasterAddr     : BYTE;
    IdentNumber    : WORD;
    ExtDiagLen     : INT;
    FaultDetected  : BOOL;
END_VAR

VAR
    ReadTrigger     : BOOL := FALSE;
    InternalDone    : BOOL := FALSE;
    DiagBuffer      : ARRAY[0..31] OF BYTE;  // Raw diagnostic data
    RequestPending  : BOOL := FALSE;

    // Simulated FB (vendor-provided block to get diagnostic data)
    FUNCTION_BLOCK DP_READ_DIAG
    VAR_INPUT
        ENABLE    : BOOL;
        SLAVE_ID  : BYTE;
    END_VAR
    VAR_OUTPUT
        DONE      : BOOL;
        ERROR     : BOOL;
        ERROR_ID  : WORD;
        DATA      : ARRAY[0..31] OF BYTE;
    END_VAR
END_VAR

VAR
    DiagFB : DP_READ_DIAG;
END_VAR
