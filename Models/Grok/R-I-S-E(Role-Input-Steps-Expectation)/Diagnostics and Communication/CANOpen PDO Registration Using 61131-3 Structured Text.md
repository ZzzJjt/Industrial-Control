FUNCTION_BLOCK CAN_REGISTER_COBID
VAR_INPUT
    EXECUTE: BOOL;          // Trigger execution on rising edge
    COBID: DWORD;           // CAN Object Identifier (0x1–0x7FF, 0 for clear)
    REGISTER: BOOL;         // TRUE to register, FALSE to deregister
END_VAR

VAR_OUTPUT
    DONE: BOOL;             // TRUE when operation completes successfully
    ERROR: BOOL;            // TRUE if error occurs
    ERROR_CODE: INT;        // 0: No error, 1: Invalid COBID, 2: Duplicate, 3: Buffer full, 4: Internal fault
END_VAR

VAR
    Buffer: ARRAY[1..32] OF DWORD; // Reception buffer for registered COBIDs
    BufferCount: INT := 0;         // Number of registered COBIDs
    ExecuteEdge: BOOL;             // Edge detection for EXECUTE
    i: INT;                        // Loop variable
    Found: BOOL;                   // Flag for COBID search
END_VAR

// Reset outputs when not executing
IF NOT EXECUTE THEN
    DONE := FALSE;
    ERROR := FALSE;
    ERROR_CODE := 0;
    ExecuteEdge := FALSE;
    RETURN;
END_IF;

// Detect rising edge of EXECUTE
IF EXECUTE AND NOT ExecuteEdge THEN
    ExecuteEdge := TRUE;
    
    // Reset outputs
    DONE := FALSE;
    ERROR := FALSE;
    ERROR_CODE := 0;
    
    // Special case: Clear all registrations and buffer
    IF NOT REGISTER AND COBID = 0 THEN
        FOR i := 1 TO 32 DO
            Buffer[i] := 0;
        END_FOR;
        BufferCount := 0;
        DONE := TRUE;
        RETURN;
    END_IF;
    
    // Validate COBID (0x1–0x7FF for standard CAN)
    IF COBID < 16#1 OR COBID > 16#7FF THEN
        ERROR := TRUE;
        ERROR_CODE := 1; // Invalid COBID
        RETURN;
    END_IF;
    
    // Check for internal buffer consistency
    IF BufferCount < 0 OR BufferCount > 32 THEN
        ERROR := TRUE;
        ERROR_CODE := 4; // Internal fault
        RETURN;
    END_IF;
    
    // Register COBID
    IF REGISTER THEN
        // Check for duplicate
        Found := FALSE;
        FOR i := 1 TO BufferCount DO
            IF Buffer[i] = COBID THEN
                Found := TRUE;
                EXIT;
            END_IF;
        END_FOR;
        
        IF Found THEN
            ERROR := TRUE;
            ERROR_CODE := 2; // Duplicate registration
            RETURN;
        END_IF;
        
        // Check buffer capacity
        IF BufferCount >= 32 THEN
            ERROR := TRUE;
            ERROR_CODE := 3; // Buffer full
            RETURN;
        END_IF;
        
        // Register COBID
        BufferCount := BufferCount + 1;
        Buffer[BufferCount] := COBID;
        DONE := TRUE;
    
    // Deregister COBID
    ELSE
        Found := FALSE;
        FOR i := 1 TO BufferCount DO
            IF Buffer[i] = COBID THEN
                Found := TRUE;
                // Shift remaining entries
                FOR j := i TO BufferCount-1 DO
                    Buffer[j] := Buffer[j+1];
                END_FOR;
                Buffer[BufferCount] := 0;
                BufferCount := BufferCount - 1;
                EXIT;
            END_IF;
        END_FOR;
        
        IF NOT Found THEN
            ERROR := TRUE;
            ERROR_CODE := 1; // Invalid COBID (not found)
            RETURN;
        END_IF;
        
        DONE := TRUE;
    END_IF;
ELSE
    // Clear DONE and ERROR on falling edge or sustained execution
    IF ExecuteEdge THEN
        DONE := FALSE;
        ERROR := FALSE;
        ERROR_CODE := 0;
    END_IF;
END_IF;
END_FUNCTION_BLOCK
