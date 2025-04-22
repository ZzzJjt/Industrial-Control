```
FUNCTION_BLOCK ETH_POWERLINK_DIAGNOSTICS
VAR_INPUT
    MN_ADDRESS : UINT;
    NODE_ID : UINT;
VAR_OUTPUT
    COMMUNICATION_STATUS : BOOL;
    ERROR_CODE : UINT;
    HEALTH_STATUS : BOOL;
    DIAGNOSTIC_INFO : STRING;
VAR
    DIAGNOSTIC_REQUEST : BOOL := FALSE;
    DIAGNOSTIC_RESPONSE_RECEIVED : BOOL := FALSE;
    DIAGNOSTIC_RESPONSE : STRUCT
        COMMUNICATION_STATUS : BOOL;
        ERROR_CODE : UINT;
        HEALTH_STATUS : BOOL;
        INFO : STRING;
    END_STRUCT;
    ERROR_LOG : ARRAY [0..100] OF STRUCT TIME : TIME; ERROR_CODE : UINT; MESSAGE : STRING; END_STRUCT;
    DIAGNOSTIC_INTERVAL : TIME := T#5s;
    TIMER : TON;
END_VAR

// Initialize timer for periodic diagnostics
TIMER(IN := TRUE, PT := DIAGNOSTIC_INTERVAL);

// Request diagnostics every DIAGNOSTIC_INTERVAL
IF TIMER.Q THEN
    DIAGNOSTIC_REQUEST := TRUE;
    TIMER(IN := FALSE); // Reset timer after request
END_IF

// Send diagnostic request to MN
IF DIAGNOSTIC_REQUEST THEN
    SEND_DIAGNOSTIC_REQUEST(MN_ADDRESS, NODE_ID);
    DIAGNOSTIC_REQUEST := FALSE;
END_IF

// Receive diagnostic response from MN
IF DIAGNOSTIC_RESPONSE_RECEIVED THEN
    COMMUNICATION_STATUS := DIAGNOSTIC_RESPONSE.COMMUNICATION_STATUS;
    ERROR_CODE := DIAGNOSTIC_RESPONSE.ERROR_CODE;
    HEALTH_STATUS := DIAGNOSTIC_RESPONSE.HEALTH_STATUS;
    DIAGNOSTIC_INFO := DIAGNOSTIC_RESPONSE.INFO;
    
    // Log error if present
    IF ERROR_CODE > 0 THEN
        ERROR_LOG[0].TIME := T#NOW;
        ERROR_LOG[0].ERROR_CODE := ERROR_CODE;
        ERROR_LOG[0].MESSAGE := DIAGNOSTIC_INFO;
        
        // Shift older logs
        FOR i := 99 Downto 1 LOOP
            ERROR_LOG[i] := ERROR_LOG[i-1];
        END_FOR
    END_IF
    
    DIAGNOSTIC_RESPONSE_RECEIVED := FALSE;
END_IF

// Reset timer if communication fails
IF NOT COMMUNICATION_STATUS THEN
    TIMER(IN := TRUE);
END_IF

// Example pseudo-functions for demonstration
PROCEDURE SEND_DIAGNOSTIC_REQUEST(MN_ADDR : UINT; NODE_ID : UINT)
    // Actual implementation depends on the PowerLink library used
    // This function sends a request to the MN to get diagnostics from the specified node
END_PROC

PROCEDURE RECEIVE_DIAGNOSTIC_RESPONSE(DIAG_RESP : REFERENCE TO STRUCT)
    // Actual implementation depends on the PowerLink library used
    // This function receives the diagnostic response from the MN and populates the DIAG_RESP structure
    DIAG_RESP.COMMUNICATION_STATUS := TRUE;
    DIAG_RESP.ERROR_CODE := 0;
    DIAG_RESP.HEALTH_STATUS := TRUE;
    DIAG_RESP.INFO := "Node is healthy.";
END_PROC
END_FUNCTION_BLOCK
```
