FUNCTION_BLOCK FB_ProfibusDP_Diagnostics
VAR_INPUT
    Execute : BOOL;                     // Rising edge triggers diagnostics
    SlaveAddress : BYTE := 1;           // Target DP slave address (1-126)
    Timeout : TIME := T#1S;             // Max wait time for response
    CyclicInterval : TIME := T#10S;     // Automatic refresh interval (0=disabled)
END_VAR

VAR_OUTPUT
    Done : BOOL;                        // Diagnostics completed successfully
    Busy : BOOL;                        // Operation in progress
    Error : BOOL;                       // Error occurred
    ErrorID : WORD;                     // Detailed error code
    DeviceStatus : WORD;                // DP slave status word
    ModuleStatus : ARRAY[1..8] OF BYTE; // Slot/diagnostic status bytes
    LastDiagTime : DATE_AND_TIME;       // Timestamp of last successful read
END_VAR

VAR
    {attribute 'hidden'}
    internalState : INT := 0;
    {attribute 'hidden'}
    prevExecute : BOOL;
    {attribute 'hidden'}
    cyclicTimer : TON;
    {attribute 'hidden'}
    responseTimer : TON;
    {attribute 'hidden'}
    diagData : ARRAY[0..31] OF BYTE;    // Raw diagnostic buffer
END_VAR

METHOD MonitorSlave : BOOL
VAR_TEMP
    cmdStatus : BOOL;
    edgeTrigger : BOOL;
END_VAR

edgeTrigger := Execute AND NOT prevExecute;
prevExecute := Execute;

// Cyclic execution logic
IF CyclicInterval > T#0S THEN
    cyclicTimer(IN:=NOT Busy, PT:=CyclicInterval);
    edgeTrigger := edgeTrigger OR cyclicTimer.Q;
END_IF

CASE internalState OF
    0: // Idle
        IF edgeTrigger THEN
            Busy := TRUE;
            Done := FALSE;
            Error := FALSE;
            responseTimer(IN:=TRUE, PT:=Timeout);
            internalState := 10;
        END_IF
        
    10: // Initiate diagnostic request
        cmdStatus := DP_SEND_DIAG(SlaveAddr:=SlaveAddress, 
                                 Buffer:=ADR(diagData), 
                                 Size:=SIZEOF(diagData));
        
        IF NOT cmdStatus THEN
            Error := TRUE;
            ErrorID := 16#8001; // Communication error
            internalState := 90;
        ELSIF responseTimer.Q THEN
            Error := TRUE;
            ErrorID := 16#8002; // Timeout
            internalState := 90;
        ELSE
            internalState := 20;
        END_IF
        
    20: // Check response completion
        IF DP_DIAG_COMPLETE(SlaveAddr:=SlaveAddress) THEN
            internalState := 30;
        ELSIF responseTimer.Q THEN
            Error := TRUE;
            ErrorID := 16#8002; // Timeout
            internalState := 90;
        END_IF
        
    30: // Parse diagnostic data
        DeviceStatus := WORD#16#0000;
        DeviceStatus := SHL(WORD(diagData[1]), 8) OR WORD(diagData[0]);
        
        // Extract module status (example for 8 slots)
        FOR i := 1 TO 8 DO
            ModuleStatus[i] := diagData[2 + i];
        END_FOR
        
        LastDiagTime := DT_NOW();
        Done := TRUE;
        internalState := 0;
        
    90: // Error handling
        Busy := FALSE;
        internalState := 0;
END_CASE

METHOD DeviceFaultActive : BOOL
// Bit 3 of status word indicates device fault
DeviceFaultActive := (DeviceStatus AND 16#0008) <> 0;
END_METHOD

METHOD CommFaultActive : BOOL
// Bit 7 of status word indicates communication fault
CommFaultActive := (DeviceStatus AND 16#0080) <> 0;
END_METHOD

METHOD GetModuleFault : BOOL
VAR_INPUT
    slot : INT; // 1-8
END_VAR
// Each bit in module status represents a channel fault
GetModuleFault := (ModuleStatus[slot] <> 0);
END_METHOD

PROGRAM MAIN
VAR
    pbDiag : FB_ProfibusDP_Diagnostics;
    alarmTrigger : BOOL;
    slave1Fault : BOOL;
END_VAR

// Cyclic monitoring of slave address 3
pbDiag(
    Execute := TRUE,  // Manual trigger still possible
    SlaveAddress := 3,
    CyclicInterval := T#5S);

// Process results
alarmTrigger := pbDiag.DeviceFaultActive OR pbDiag.CommFaultActive;

// Check module 2 status
slave1Fault := pbDiag.GetModuleFault(slot := 2);

// Error handling
IF pbDiag.Error THEN
    CASE pbDiag.ErrorID OF
        16#8001: // Handle comm error
        16#8002: // Handle timeout
    END_CASE
END_IF

