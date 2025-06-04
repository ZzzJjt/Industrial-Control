FUNCTION_BLOCK FB_ProfibusDPV1_Diagnostics
VAR_INPUT
    Execute : BOOL;
    DiagnosticType : BYTE; // Diagnostic code received from Profibus device
    RawData : ARRAY[1..20] OF BYTE; // Example raw diagnostic buffer
END_VAR

VAR_OUTPUT
    Done : BOOL;
    Error : BOOL;
    ErrorID : WORD;
    CommErrorFlag : BOOL;
    DeviceStatusFlag : BOOL;
    ParamErrorFlag : BOOL;
    WatchdogFlag : BOOL;
    ConfigMismatchFlag : BOOL;
    PowerFaultFlag : BOOL;
    HWFailureFlag : BOOL;
    BusInterruptFlag : BOOL;
    TempLimitFlag : BOOL;
    VendorDiagFlag : BOOL;
END_VAR

VAR
    tmpErrorID : WORD;
END_VAR

IF Execute THEN
    Error := FALSE;
    Done := FALSE;

    CASE DiagnosticType OF
        16#01: // Communication Error
            CommErrorFlag := TRUE;
            tmpErrorID := SHL(WORD(RawData[2]), 8) OR WORD(RawData[3]);

        16#02: // Device Status
            DeviceStatusFlag := TRUE;

        16#03: // Parameter Consistency
            ParamErrorFlag := TRUE;

        16#04: // Watchdog Timeout
            WatchdogFlag := TRUE;

        16#05: // Configuration Mismatch
            ConfigMismatchFlag := TRUE;

        16#06: // Power Supply Fault
            PowerFaultFlag := TRUE;

        16#07: // Hardware Failure
            HWFailureFlag := TRUE;

        16#08: // Bus Interruption
            BusInterruptFlag := TRUE;

        16#09: // Temperature Warning
            TempLimitFlag := TRUE;

        16#0A: // Manufacturer Specific Diagnostic
            VendorDiagFlag := TRUE;

        ELSE
            Error := TRUE;
            tmpErrorID := 999; // Unknown diagnostic
    END_CASE;

    ErrorID := tmpErrorID;
    IF NOT Error THEN
        Done := TRUE;
    END_IF
END_IF
