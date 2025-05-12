FUNCTION_BLOCK CAN_REGISTER_COBID
VAR_INPUT
    REGISTER : BOOL;           // TRUE to register, FALSE to deregister
    COBID : UDINT;             // CAN Object ID to register (0 means all when deregistering)
    EXECUTE : BOOL := FALSE;   // Rising edge triggers operation
END_VAR

VAR_OUTPUT
    BUSY : BOOL := FALSE;      // Operation in progress
    DONE : BOOL := FALSE;      // Operation completed successfully
    ERROR : BOOL := FALSE;     // Error occurred
    ERROR_CODE : UINT := 16#0000; // Detailed error code
END_VAR

VAR
    // Internal state tracking
    ExecutePrev : BOOL := FALSE;
    OperationActive : BOOL := FALSE;
    
    // Registered COBIDs tracking
    RegisteredCOBIDs : ARRAY[1..MAX_COBIDS] OF UDINT;
    RegistrationCount : UINT := 0;
    
    // Constants
    MAX_COBIDS : UINT := 32;   // Maximum number of COBIDs we can track
    CANOPEN_SUCCESS : UINT := 16#0000;
END_VAR

METHOD RegisterSingleCOBID : UINT
VAR_INPUT
    id : UDINT;
END_VAR
VAR
    status : UINT;
    i : UINT;
END_VAR
BEGIN
    // Check if COBID is already registered
    FOR i := 1 TO RegistrationCount DO
        IF RegisteredCOBIDs[i] = id THEN
            RETURN 16#8001; // Duplicate registration error
        END_IF
    END_FOR
    
    // Check if we have space for new registration
    IF RegistrationCount >= MAX_COBIDS THEN
        RETURN 16#8002; // Maximum registrations exceeded
    END_IF
    
    // Validate COBID format (CANOpen specific rules)
    IF (id AND 16#80000000) <> 0 THEN
        RETURN 16#8003; // Invalid COBID format
    END_IF
    
    // Call low-level CAN layer to register COBID
    status := CAN_LAYER_REGISTER(id);
    IF status <> CANOPEN_SUCCESS THEN
        RETURN status; // Return CAN layer error code
    END_IF
    
    // Add to our tracking array
    RegistrationCount := RegistrationCount + 1;
    RegisteredCOBIDs[RegistrationCount] := id;
    
    RETURN CANOPEN_SUCCESS;
END_METHOD

METHOD DeregisterSingleCOBID : UINT
VAR_INPUT
    id : UDINT;
END_VAR
VAR
    status : UINT;
    i, j : UINT;
    found : BOOL := FALSE;
END_VAR
BEGIN
    // Find and remove the COBID from our tracking
    FOR i := 1 TO RegistrationCount DO
        IF RegisteredCOBIDs[i] = id THEN
            found := TRUE;
            // Shift remaining entries down
            FOR j := i TO RegistrationCount-1 DO
                RegisteredCOBIDs[j] := RegisteredCOBIDs[j+1];
            END_FOR
            RegistrationCount := RegistrationCount - 1;
            EXIT;
        END_IF
    END_FOR
    
    IF NOT found THEN
        RETURN 16#8004; // COBID not registered
    END_IF
    
    // Call low-level CAN layer to deregister
    status := CAN_LAYER_DEREGISTER(id);
    IF status <> CANOPEN_SUCCESS THEN
        RETURN status;
    END_IF
    
    RETURN CANOPEN_SUCCESS;
END_METHOD

METHOD ClearAllRegistrations : UINT
VAR
    status : UINT;
    i : UINT;
END_VAR
BEGIN
    // Deregister all known COBIDs
    FOR i := 1 TO RegistrationCount DO
        status := CAN_LAYER_DEREGISTER(RegisteredCOBIDs[i]);
        IF status <> CANOPEN_SUCCESS THEN
            // Continue deregistering others but return last error
            ERROR_CODE := status;
        END_IF
    END_FOR
    
    // Clear network layer buffer
    status := CAN_LAYER_CLEAR_BUFFER();
    IF status <> CANOPEN_SUCCESS THEN
        RETURN status;
    END_IF
    
    // Reset our tracking
    RegistrationCount := 0;
    
    RETURN CANOPEN_SUCCESS;
END_METHOD

// Main function block logic
BEGIN
    // Reset outputs on each execution cycle
    DONE := FALSE;
    ERROR := FALSE;
    
    // Detect rising edge of EXECUTE
    IF EXECUTE AND NOT ExecutePrev AND NOT OperationActive THEN
        OperationActive := TRUE;
        BUSY := TRUE;
        
        // Reset error state
        ERROR_CODE := 16#0000;
    END_IF
    
    ExecutePrev := EXECUTE;
    
    // Handle active operation
    IF OperationActive THEN
        IF REGISTER THEN
            // Registration request
            IF COBID = 0 THEN
                ERROR_CODE := 16#8005; // Invalid COBID for registration
                ERROR := TRUE;
            ELSE
                ERROR_CODE := RegisterSingleCOBID(COBID);
                ERROR := ERROR_CODE <> CANOPEN_SUCCESS;
            END_IF
        ELSE
            // Deregistration request
            IF COBID = 0 THEN
                // Full reset case
                ERROR_CODE := ClearAllRegistrations();
                ERROR := ERROR_CODE <> CANOPEN_SUCCESS;
            ELSE
                // Single deregistration
                ERROR_CODE := DeregisterSingleCOBID(COBID);
                ERROR := ERROR_CODE <> CANOPEN_SUCCESS;
            END_IF
        END_IF
        
        // Operation complete
        OperationActive := FALSE;
        BUSY := FALSE;
        DONE := TRUE;
    END_IF
END_FUNCTION_BLOCK
