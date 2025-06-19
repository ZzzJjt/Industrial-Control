FUNCTION_BLOCK FB_DPV1DiagHandler
VAR_INPUT
    Execute       : BOOL;      // Trigger to start diagnostic read
    SlaveAddress  : BYTE;      // Target Profibus DPV1 slave
    Timeout       : TIME;      // Max wait time
END_VAR
VAR_OUTPUT
    Done          : BOOL;
    Busy          : BOOL;
    Error         : BOOL;
    ErrorID       : DWORD;

    // Diagnostic Results
    CommError         : BOOL;
    DeviceFailure     : BOOL;
    ParamFault        : BOOL;
    HardwareWarning   : BOOL;
    VendorDiagCode    : WORD;
END_VAR
VAR
    StartTime      : TIME;
    CurrentTime    : TIME;
    DiagRequestSent : BOOL;
    RisingEdge     : R_TRIG;

    DiagTypeID     : BYTE;
    DiagData       : ARRAY[0..15] OF BYTE;

    InternalState  : INT;
END_VAR

RisingEdge(CLK := Execute);

CASE InternalState OF
0: // Idle
    IF RisingEdge.Q THEN
        Busy := TRUE;
        Done := FALSE;
        Error := FALSE;
        ErrorID := 0;
        DiagRequestSent := FALSE;
        StartTime := TIME();
        InternalState := 1;
    END_IF

1: // Send request
    // Simulate request for this example
    DiagRequestSent := TRUE;
    InternalState := 2;

2: // Wait for response
    CurrentTime := TIME();
    IF (CurrentTime - StartTime) > Timeout THEN
        Error := TRUE;
        ErrorID := 16#00020001; // Timeout
        Busy := FALSE;
        InternalState := 0;
    ELSIF DiagRequestSent THEN
        // Simulated diagnostic block
        DiagTypeID := DiagData[0];  // Diag type in first byte
        InternalState := 3;
    END_IF

3: // Process diagnostic types
    CASE DiagTypeID OF
        1: // Communication error
            CommError := TRUE;

        2: // Device failure
            DeviceFailure := TRUE;

        3: // Parameter fault
            ParamFault := TRUE;

        4: // Hardware warning
            HardwareWarning := TRUE;

        5: // Vendor-specific diagnostic
            VendorDiagCode := WORD_TO_WORD(SHL(DiagData[1], 8) OR DiagData[2]);

        6: // Power Supply Alert
            HardwareWarning := TRUE;

        7: // Config Mismatch
            ParamFault := TRUE;

        8: // EEPROM Error
            DeviceFailure := TRUE;

        9: // Watchdog Triggered
            CommError := TRUE;

        10: // Overtemperature
            HardwareWarning := TRUE;

        ELSE
            Error := TRUE;
            ErrorID := 16#00020002; // Unknown Diag Type
    END_CASE;

    Done := TRUE;
    Busy := FALSE;
    InternalState := 0;
END_CASE
