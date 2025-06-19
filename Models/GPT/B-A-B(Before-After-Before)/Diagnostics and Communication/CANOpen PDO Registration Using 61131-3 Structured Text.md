FUNCTION_BLOCK CAN_REGISTER_COBID
VAR_INPUT
    COBID      : DWORD;   // COB-ID to register/deregister (0 = clear all if deregistering)
    REGISTER   : BOOL;    // TRUE = register, FALSE = deregister
    EXECUTE    : BOOL;    // Rising edge triggers execution
END_VAR

VAR_OUTPUT
    STATUS     : INT;     // 0 = OK, 1 = Invalid COBID, 2 = Already Registered, 3 = Not Registered, -1 = Comm Error
    DONE       : BOOL;    // TRUE on successful operation
    ERROR      : BOOL;    // TRUE if STATUS ≠ 0
END_VAR

VAR
    COBID_LIST : ARRAY[1..10] OF DWORD := [0,0,0,0,0,0,0,0,0,0]; // Registered COBID buffer
    i          : INT;
    found      : BOOL;
    emptySlot  : INT := 0;
    triggered  : BOOL;
END_VAR

// Rising edge detection
IF EXECUTE AND NOT triggered THEN
    triggered := TRUE;
    DONE := FALSE;
    ERROR := FALSE;
    STATUS := 0;
    found := FALSE;

    // Validate COBID (basic check)
    IF (COBID > 16#7FF) AND (COBID < 16#80000000) THEN
        STATUS := 1; // Invalid COBID range
        ERROR := TRUE;
        RETURN;
    END_IF

    // Handle full clear operation
    IF NOT REGISTER AND COBID = 0 THEN
        FOR i := 1 TO 10 DO
            COBID_LIST[i] := 0;
        END_FOR
        // Purge network buffer logic would go here
        DONE := TRUE;
        RETURN;
    END_IF

    // Search for COBID in list
    FOR i := 1 TO 10 DO
        IF COBID_LIST[i] = COBID THEN
            found := TRUE;
            EXIT;
        ELSIF (COBID_LIST[i] = 0) AND (emptySlot = 0) THEN
            emptySlot := i;
        END_IF
    END_FOR

    // REGISTER = TRUE: try to register new COBID
    IF REGISTER THEN
        IF found THEN
            STATUS := 2; // Already registered
            ERROR := TRUE;
        ELSIF emptySlot > 0 THEN
            COBID_LIST[emptySlot] := COBID;
            // Call CAN layer registration here
            DONE := TRUE;
        ELSE
            STATUS := -1; // No space left or comm error
            ERROR := TRUE;
        END_IF

    // REGISTER = FALSE: try to deregister COBID
    ELSE
        IF found THEN
            COBID_LIST[i] := 0;
            // Call CAN layer deregistration here
            DONE := TRUE;
        ELSE
            STATUS := 3; // Not registered
            ERROR := TRUE;
        END_IF
    END_IF

ELSE
    triggered := FALSE; // Reset edge detection
END_IF
