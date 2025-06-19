TYPE DPV1_DiagType :
STRUCT
    CommunicationError : WORD;       // Error code + telegram info
    DeviceStatus : WORD;             // Bitmasked status flags
    ParameterFault : ARRAY[1..4] OF BYTE;  // Faulty parameter IDs
    ConfigMismatch : DWORD;          // Expected vs actual config
    VoltageStatus : STRUCT           // Power supply monitoring
        MainVoltage : BOOL;
        BackupVoltage : BOOL;
        UndervoltageCount : UINT;
    END_STRUCT;
    Temperature : STRUCT
        OverTemp : BOOL;
        CurrentTemp : INT;
        Threshold : INT;
    END_STRUCT;
    HardwareFault : WORD;            // Bitmasked HW faults
    BusStatus : STRUCT
        Interrupted : BOOL;
        RetryCount : UINT;
        LastErrorCode : BYTE;
    END_STRUCT;
    Watchdog : STRUCT
        Timeout : BOOL;
        Interval : UINT;
        LastResetTime : UDINT;
    END_STRUCT;
    VendorSpecific : ARRAY[0..7] OF BYTE;  // Custom manufacturer data
END_STRUCT
END_TYPE

FUNCTION_BLOCK FB_DPV1_DiagnosticParser
VAR_INPUT
    Execute : BOOL;                     // Trigger diagnostic read
    SlaveAddress : BYTE := 1;           // DP slave address (1-126)
    Timeout : TIME := T#500MS;          // Response timeout
END_VAR

VAR_OUTPUT
    DiagData : DPV1_DiagType;           // Structured diagnostics
    NewData : BOOL;                     // Fresh data available
    Error : BOOL;                       // Read/parse error occurred
    ErrorCode : WORD;                   // Detailed error information
END_VAR

VAR
    {attribute 'hidden'}
    internalState : INT := 0;
    {attribute 'hidden'}
    prevExecute : BOOL;
    {attribute 'hidden'}
    diagBuffer : ARRAY[0..63] OF BYTE;  // Raw diagnostic data
    {attribute 'hidden'}
    timer : TON;
END_VAR

METHOD ParseDiagnostics : BOOL
VAR_TEMP
    diagType : BYTE;
    dataLength : UINT;
    parseOK : BOOL;
END_VAR

CASE internalState OF
    0: // Idle
        IF Execute AND NOT prevExecute THEN
            timer(IN:=TRUE, PT:=Timeout);
            internalState := 10;
        END_IF
        prevExecute := Execute;
        
    10: // Read diagnostic data
        IF NOT DPV1_ReadDiagnostics(SlaveAddress, ADR(diagBuffer), SIZEOF(diagBuffer)) THEN
            Error := TRUE;
            ErrorCode := 16#8001;
            internalState := 90;
        ELSIF timer.Q THEN
            Error := TRUE;
            ErrorCode := 16#8002;
            internalState := 90;
        ELSE
            internalState := 20;
        END_IF
        
    20: // Verify response
        IF DPV1_DiagComplete(SlaveAddress) THEN
            internalState := 30;
        ELSIF timer.Q THEN
            Error := TRUE;
            ErrorCode := 16#8002;
            internalState := 90;
        END_IF
        
    30: // Parse header
        diagType := diagBuffer[0];
        dataLength := UINT(diagBuffer[1]) * 256 + UINT(diagBuffer[2]);
        
        // Validate message length
        IF dataLength > (SIZEOF(diagBuffer) - 3) THEN
            Error := TRUE;
            ErrorCode := 16#8003;
            internalState := 90;
        ELSE
            internalState := 40;
        END_IF
        
    40: // Type-specific parsing
        CASE diagType OF
            1:  // Communication errors
                DiagData.CommunicationError := WORD(diagBuffer[3]) * 256 + WORD(diagBuffer[4]);
                parseOK := TRUE;
                
            2:  // Device status
                DiagData.DeviceStatus := WORD(diagBuffer[3]) * 256 + WORD(diagBuffer[4]);
                parseOK := TRUE;
                
            3:  // Parameter faults
                FOR i := 1 TO 4 DO
                    DiagData.ParameterFault[i] := diagBuffer[3 + i];
                END_FOR
                parseOK := TRUE;
                
            // Additional cases for other diagnostic types...
            10: // Manufacturer-specific
                FOR i := 0 TO 7 DO
                    DiagData.VendorSpecific[i] := diagBuffer[3 + i];
                END_FOR
                parseOK := TRUE;
                
            ELSE
                Error := TRUE;
                ErrorCode := 16#8004; // Unknown diagnostic type
                parseOK := FALSE;
        END_CASE
        
        IF parseOK THEN
            NewData := TRUE;
            internalState := 0;
        ELSE
            internalState := 90;
        END_IF
        
    90: // Error handling
        NewData := FALSE;
        internalState := 0;
END_CASE

METHOD CheckVoltageFault : BOOL
// Returns TRUE if any voltage issue detected
CheckVoltageFault := NOT DiagData.VoltageStatus.MainVoltage 
                  OR (DiagData.VoltageStatus.UndervoltageCount > 0);
END_METHOD

METHOD GetCriticalFaults : WORD
// Returns bitmask of critical faults
VAR_TEMP
    result : WORD := 0;
END_VAR

// Bit 0: Communication fault
IF (DiagData.CommunicationError AND 16#F000) <> 0 THEN
    result := result OR 1;
END_IF

// Bit 1: Over temperature
IF DiagData.Temperature.OverTemp THEN
    result := result OR 2;
END_IF

// Bit 2: Hardware failure
IF (DiagData.HardwareFault AND 16#000F) <> 0 THEN
    result := result OR 4;
END_IF

RETURN result;
END_METHOD

PROGRAM MAIN
VAR
    dpv1Diag : FB_DPV1_DiagnosticParser;
    faultHandling : BOOL;
    maintenanceAlert : BOOL;
    diagCounter : UINT;
END_VAR

// Execute diagnostic read every 2 seconds
dpv1Diag(
    Execute := TRUE, 
    SlaveAddress := 5, 
    Timeout := T#1S);

// Process results
IF dpv1Diag.NewData THEN
    diagCounter := diagCounter + 1;
    
    // Check for critical conditions
    faultHandling := dpv1Diag.DiagData.BusStatus.Interrupted 
                  OR (dpv1Diag.GetCriticalFaults() <> 0);
    
    // Log temperature warnings
    IF dpv1Diag.DiagData.Temperature.OverTemp THEN
        maintenanceAlert := TRUE;
    END_IF;
END_IF

IF dpv1Diag.Error THEN
    // Handle errors according to ErrorCode
END_IF

