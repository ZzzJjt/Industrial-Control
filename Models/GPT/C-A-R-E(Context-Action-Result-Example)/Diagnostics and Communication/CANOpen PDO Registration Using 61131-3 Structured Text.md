FUNCTION_BLOCK CAN_REGISTER_COBID
VAR_INPUT
    COBID      : DWORD;     // COB-ID to (de)register, 0 = clear all
    REGISTER   : BOOL;      // TRUE = register, FALSE = deregister
END_VAR
VAR_OUTPUT
    STATUS     : INT;       // 0 = OK, 1 = Already registered, 2 = Not registered, 3 = Invalid COBID
    ERROR      : BOOL;      // TRUE = error occurred
END_VAR
VAR
    i          : INT;
    found      : BOOL;
    index      : INT;
    buffer     : ARRAY[1..10] OF DWORD; // Registered COBID storage
END_VAR

// Default outputs
STATUS := 0;
ERROR := FALSE;

// Validate input
IF COBID > 16#7FF THEN
    STATUS := 3; // Invalid standard CAN COBID (>11 bits)
    ERROR := TRUE;
    RETURN;
END_IF

// Deregistration logic
IF NOT REGISTER THEN
    IF COBID = 0 THEN
        // Clear all registrations
        FOR i := 1 TO 10 DO
            buffer[i] := 0;
        END_FOR;
        STATUS := 0;
        RETURN;
    END_IF

    // Find and remove COBID
    found := FALSE;
    FOR i := 1 TO 10 DO
        IF buffer[i] = COBID THEN
            buffer[i] := 0;
            found := TRUE;
            EXIT;
        END_IF
    END_FOR;

    IF NOT found THEN
        STATUS := 2; // Not registered
        ERROR := TRUE;
    ELSE
        STATUS := 0; // Success
    END_IF
    RETURN;
END_IF

// Registration logic
found := FALSE;
FOR i := 1 TO 10 DO
    IF buffer[i] = COBID THEN
        STATUS := 1; // Already registered
        ERROR := TRUE;
        RETURN;
    END_IF
END_FOR;

// Find free slot
FOR i := 1 TO 10 DO
    IF buffer[i] = 0 THEN
        buffer[i] := COBID;
        STATUS := 0; // Success
        RETURN;
    END_IF
END_FOR;

// No available slot
STATUS := 3; // Buffer full or invalid
ERROR := TRUE;
