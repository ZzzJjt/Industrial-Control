FUNCTION_BLOCK FB_PowerLinkDiagnostics  
VAR_INPUT  
    Enable : BOOL := TRUE;                     // Enable/disable diagnostics  
    PollingInterval : TIME := T#1S;            // Cyclic polling interval  
    NodeCount : UINT := 1;                     // Number of PowerLink nodes to monitor  
    NodeID_List : ARRAY[1..MAX_NODES] OF UINT; // List of Node IDs to monitor  
END_VAR  

VAR_OUTPUT  
    NodeStatus : ARRAY[1..MAX_NODES] OF UINT;  // Status codes per node  
    ErrorFlags : ARRAY[1..MAX_NODES] OF WORD;  // Detailed error flags  
    CommHealth : REAL;                         // Overall network health (0-100%)  
    DiagnosticsReady : BOOL := FALSE;          // Data available flag  
    Error : BOOL := FALSE;                     // Global error indicator  
    ErrorCode : UINT := 0;                     // Diagnostic error code  
END_VAR  

VAR  
    // Internal state tracking  
    PollTimer : TON;  
    CurrentNodeIndex : UINT := 1;  
    RetryCount : UINT := 0;  
    MN_Interface : MN_DIAG_Interface;          // PowerLink Managing Node interface  
    
    // Constants  
    MAX_NODES : UINT := 32;                    // Max supported nodes  
    MAX_RETRIES : UINT := 3;                   // Max retries on comm failure  
    
    // Diagnostic register addresses (adjust per PowerLink spec)  
    DIAG_STATUS_REG : UDINT := 16#F000;        // Node status register  
    DIAG_ERROR_REG : UDINT := 16#F002;         // Error flags register  
END_VAR  

// -------------------------  
// **Helper Methods**  
// -------------------------  

// Read PowerLink node diagnostic data  
METHOD ReadNodeDiagnostics : BOOL  
VAR_INPUT  
    NodeID : UINT;  
END_VAR  
VAR  
    StatusCode : UINT;  
    ErrorWord : WORD;  
    Success : BOOL;  
END_VAR  
BEGIN  
    // Read status register (e.g., via SDO or direct memory access)  
    Success := MN_Interface.ReadRegister(  
        NodeID := NodeID,  
        Address := DIAG_STATUS_REG,  
        Value => StatusCode  
    );  
    
    IF NOT Success THEN  
        Error := TRUE;  
        ErrorCode := 16#8001; // Read failure  
        RETURN FALSE;  
    END_IF;  
    
    // Read error flags if status indicates an issue  
    IF (StatusCode AND 16#0001) <> 0 THEN  
        Success := MN_Interface.ReadRegister(  
            NodeID := NodeID,  
            Address := DIAG_ERROR_REG,  
            Value => ErrorWord  
        );  
        
        IF NOT Success THEN  
            Error := TRUE;  
            ErrorCode := 16#8002; // Error register read failure  
            RETURN FALSE;  
        END_IF;  
    ELSE  
        ErrorWord := 0; // No errors  
    END_IF;  
    
    // Update outputs  
    NodeStatus[NodeID] := StatusCode;  
    ErrorFlags[NodeID] := ErrorWord;  
    
    RETURN TRUE;  
END_METHOD  

// Calculate overall network health (%)  
METHOD CalculateNetworkHealth : REAL  
VAR  
    HealthyNodes : UINT := 0;  
    i : UINT;  
END_VAR  
BEGIN  
    FOR i := 1 TO NodeCount DO  
        IF (NodeStatus[i] AND 16#0001) = 0 THEN  
            HealthyNodes := HealthyNodes + 1;  
        END_IF;  
    END_FOR;  
    
    RETURN (REAL(HealthyNodes) / REAL(NodeCount)) * 100.0;  
END_METHOD  

// -------------------------  
// **Main Execution Logic**  
// -------------------------  
BEGIN  
    // Reset outputs  
    DiagnosticsReady := FALSE;  
    Error := FALSE;  
    
    // Cyclic polling  
    IF Enable THEN  
        PollTimer(IN := TRUE, PT := PollingInterval);  
        
        IF PollTimer.Q THEN  
            PollTimer(IN := FALSE);  
            
            // Read diagnostics for the next node (round-robin)  
            IF ReadNodeDiagnostics(NodeID_List[CurrentNodeIndex]) THEN  
                RetryCount := 0;  
                CurrentNodeIndex := CurrentNodeIndex MOD NodeCount + 1;  
                
                // Update overall health metric  
                CommHealth := CalculateNetworkHealth();  
                
                // Raise data-ready flag  
                DiagnosticsReady := TRUE;  
            ELSE  
                // Retry on failure  
                RetryCount := RetryCount + 1;  
                IF RetryCount >= MAX_RETRIES THEN  
                    Error := TRUE;  
                    ErrorCode := 16#8003; // Max retries exceeded  
                END_IF;  
            END_IF;  
        END_IF;  
    ELSE  
        PollTimer(IN := FALSE);  
    END_IF;  
END_FUNCTION_BLOCK  
