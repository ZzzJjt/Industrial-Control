FUNCTION_BLOCK CAN_REGISTER_COBID
VAR_INPUT
    COBID     : DWORD;    // CAN Object Identifier to register/deregister
    REGISTER  : BOOL;     // TRUE = Register, FALSE = Deregister
    Execute   : BOOL;     // Trigger input (rising edge triggers action)
END_VAR

VAR_OUTPUT
    Success   : BOOL;     // TRUE if operation successful
    Error     : BOOL;     // TRUE if an error occurred
    ErrorCode : INT;      // Error code: 0 = OK, 1 = Invalid COBID, 2 = Comm Fault
END_VAR

VAR
    PrevExecute     : BOOL := FALSE;
    CommStatus      : BOOL := TRUE;       // Simulated comm status (TRUE = OK)
    COBIDRegistry   : ARRAY[1..100] OF DWORD;  // Simulated registry
    i               : INT;
    FoundIndex      : INT := 0;
    FreeIndex       : INT := 0;
END_VAR

// Rising edge detection
IF Execute AND NOT PrevExecute THEN
    Success := FALSE;
    Error := FALSE;
    ErrorCode := 0;
    
    IF NOT CommStatus THEN
        Error := TRUE;
        ErrorCode := 2; // Communication fault
    ELSIF (NOT REGISTER) AND (COBID = 0) THEN
        // Clear all registrations and reset buffer
        FOR i := 1 TO 100 DO
            COBIDRegistry[i] := 0;
        END_FOR;
        Success := TRUE;
    ELSIF REGISTER THEN
        // Register COBID if not already present
        FoundIndex := 0;
        FreeIndex := 0;
        FOR i := 1 TO 100 DO
            IF (COBIDRegistry[i] = COBID) THEN
                FoundIndex := i;
                EXIT;
            ELSIF (COBIDRegistry[i] = 0) AND (FreeIndex = 0) THEN
                FreeIndex := i;
            END_IF
        END_FOR;

        IF FoundIndex = 0 THEN
            IF FreeIndex > 0 THEN
                COBIDRegistry[FreeIndex] := COBID;
                Success := TRUE;
            ELSE
                Error := TRUE;
                ErrorCode := 3; // Registry full
            END_IF
        ELSE
            Success := TRUE; // Already registered
        END_IF

    ELSIF NOT REGISTER THEN
        // Deregister specified COBID
        FoundIndex := 0;
        FOR i := 1 TO 100 DO
            IF COBIDRegistry[i] = COBID THEN
                FoundIndex := i;
                EXIT;
            END_IF
        END_FOR;

        IF FoundIndex > 0 THEN
            COBIDRegistry[FoundIndex] := 0;
            Success := TRUE;
        ELSE
            Error := TRUE;
            ErrorCode := 1; // Invalid COBID
        END_IF
    END_IF
END_IF;

PrevExecute := Execute;
