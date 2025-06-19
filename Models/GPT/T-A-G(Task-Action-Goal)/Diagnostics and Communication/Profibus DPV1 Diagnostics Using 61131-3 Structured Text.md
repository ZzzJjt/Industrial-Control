PROGRAM ProfibusDPV1_DiagnosticHandler
VAR
    Execute            : BOOL;
    SlaveAddress       : BYTE;
    DiagTypeCode       : BYTE; // Diagnostic type code (0..9)
    DiagValueRaw       : WORD;
    Done               : BOOL := FALSE;
    Error              : BOOL := FALSE;
    ErrorID            : DWORD := 0;

    CommError          : BOOL;
    DeviceStatus       : BYTE;
    ParamFault         : BOOL;
    ConfigIssue        : BOOL;
    PowerSupplyFail    : BOOL;
    HardwareFail       : BOOL;
    BusInterrupt       : BOOL;
    WatchdogTimeout    : BOOL;
    TempAlert          : BOOL;
    VendorDiagCode     : WORD;
END_VAR

// Simulated diagnostic fetch call
DiagValueRaw := GetDPV1Diagnostic(SlaveAddress, DiagTypeCode); // returns WORD

CASE DiagTypeCode OF

    0: // Communication Error
        CommError := (DiagValueRaw AND 16#0001) <> 0;
        Done := TRUE;

    1: // Device Status
        DeviceStatus := BYTE(DiagValueRaw AND 16#00FF);
        Done := TRUE;

    2: // Parameter Fault
        ParamFault := (DiagValueRaw AND 16#0001) <> 0;
        Done := TRUE;

    3: // Configuration Issue
        ConfigIssue := (DiagValueRaw AND 16#0001) <> 0;
        Done := TRUE;

    4: // Power Supply Problem
        PowerSupplyFail := (DiagValueRaw AND 16#0001) <> 0;
        Done := TRUE;

    5: // Hardware Failure
        HardwareFail := (DiagValueRaw AND 16#0001) <> 0;
        Done := TRUE;

    6: // Bus Interruption
        BusInterrupt := (DiagValueRaw AND 16#0001) <> 0;
        Done := TRUE;

    7: // Watchdog Timeout
        WatchdogTimeout := (DiagValueRaw AND 16#0001) <> 0;
        Done := TRUE;

    8: // Temperature Alert
        TempAlert := (DiagValueRaw AND 16#0001) <> 0;
        Done := TRUE;

    9: // Vendor Specific Diagnostics
        VendorDiagCode := DiagValueRaw;
        Done := TRUE;

    ELSE
        Error := TRUE;
        ErrorID := 1001; // Unsupported diagnostic type
        Done := FALSE;

END_CASE

// Simulated stub
FUNCTION GetDPV1Diagnostic : WORD
VAR_INPUT
    addr : BYTE;
    dtype : BYTE;
END_VAR
// Simulated response generator
GetDPV1Diagnostic := 16#0001 + dtype;
