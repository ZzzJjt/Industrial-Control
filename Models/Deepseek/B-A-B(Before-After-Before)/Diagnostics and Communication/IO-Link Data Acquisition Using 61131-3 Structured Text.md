FUNCTION_BLOCK FB_IO_Link_DataAcquisition
VAR_INPUT
    Execute : BOOL := FALSE;                     // Rising edge triggers read cycle
    Timeout : TIME := T#1S;                     // Max wait time per read
    Retries : UINT := 3;                        // Max retry attempts on failure
    ChannelMap : ARRAY[1..5] OF BOOL := [TRUE, TRUE, TRUE, TRUE, TRUE]; // Enable/disable channels
END_VAR

VAR_OUTPUT
    ProcessValues : ARRAY[1..5] OF REAL := [0.0, 0.0, 0.0, 0.0, 0.0];  // Acquired values
    Status : ARRAY[1..5] OF UINT;               // Status codes per channel
    Busy : BOOL := FALSE;                       // Read operation in progress
    Done : BOOL := FALSE;                       // All reads completed
    Error : BOOL := FALSE;                      // Global error flag
    ErrorCode : UINT := 16#0000;                // Detailed error code
END_VAR

VAR
    // Internal state tracking
    ExecuteRisingEdge : BOOL := FALSE;
    ActiveChannel : UINT := 1;
    RetryCount : UINT := 0;
    ReadTimer : TON;
    
    // IO-Link master interface
    IOLinkMaster : IOL_MASTER_IFACE;            // Replace with actual IO-Link FB
    
    // Constants
    STATUS_SUCCESS : UINT := 16#0000;
    STATUS_TIMEOUT : UINT := 16#8001;
    STATUS_COMM_ERROR : UINT := 16#8002;
    STATUS_INVALID_CH : UINT := 16#8003;
END_VAR

// -------------------------
// **Helper Methods**
// -------------------------

// Read single IO-Link process value
METHOD ReadProcessValue : UINT
VAR_INPUT
    Channel : UINT;                             // 1-5
END_VAR
VAR
    Value : REAL;
    Success : BOOL;
    DiagCode : UINT;
BEGIN
    // Validate channel
    IF (Channel < 1) OR (Channel > 5) THEN
        RETURN STATUS_INVALID_CH;
    END_IF;
    
    // Read via IO-Link master interface
    Success := IOLinkMaster.ReadProcessData(
        Channel := Channel,
        Timeout := Timeout,
        Value => Value,
        Status => DiagCode
    );
    
    // Process result
    IF NOT Success THEN
        IF DiagCode = 16#0000 THEN
            DiagCode := STATUS_TIMEOUT;         // Default to timeout if no code
        END_IF;
        RETURN DiagCode;
    ELSE
        ProcessValues[Channel] := Value;        // Store successful read
        RETURN STATUS_SUCCESS;
    END_IF;
END_METHOD

// -------------------------
// **Main Execution Logic**
// -------------------------
BEGIN
    // Reset cycle flags
    Done := FALSE;
    Error := FALSE;
    
    // Detect rising edge of Execute
    ExecuteRisingEdge := Execute AND NOT ExecuteRisingEdge;
    
    // Start new read cycle
    IF ExecuteRisingEdge AND NOT Busy THEN
        Busy := TRUE;
        ActiveChannel := 1;
        RetryCount := 0;
    END_IF;
    
    // Process active read cycle
    IF Busy THEN
        // Skip disabled channels
        WHILE (ActiveChannel <= 5) AND NOT ChannelMap[ActiveChannel] DO
            ActiveChannel := ActiveChannel + 1;
        END_WHILE;
        
        // Exit if all channels processed
        IF ActiveChannel > 5 THEN
            Busy := FALSE;
            Done := TRUE;
            RETURN;
        END_IF;
        
        // Read current channel
        Status[ActiveChannel] := ReadProcessValue(ActiveChannel);
        
        // Handle read result
        IF Status[ActiveChannel] = STATUS_SUCCESS THEN
            ActiveChannel := ActiveChannel + 1;  // Proceed to next channel
            RetryCount := 0;
        ELSE
            // Retry or fail
            RetryCount := RetryCount + 1;
            IF RetryCount >= Retries THEN
                Error := TRUE;
                ErrorCode := Status[ActiveChannel];
                Busy := FALSE;                  // Abort cycle on max retries
            END_IF;
        END_IF;
    END_IF;
END_FUNCTION_BLOCK
