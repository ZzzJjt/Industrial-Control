FUNCTION_BLOCK CAN_REGISTER_COBID
VAR_INPUT
    COBID     : DWORD;    // Communication Object Identifier to register or deregister
    REGISTER  : BOOL;     // TRUE to register, FALSE to deregister
    ENABLE    : BOOL;     // Activation input for one-shot execution
END_VAR

VAR_OUTPUT
    DONE      : BOOL;     // Operation completed successfully
    ERROR     : BOOL;     // Error occurred
    ERR_CODE  : INT;      // Error code (0: OK, 1: Invalid COBID, 2: Duplicate, 3: Not Found, 9: Internal Fault)
END_VAR

VAR
    RegTable  : ARRAY [1..100] OF DWORD := [0(0)]; // Static registration table
    i         : INT;
    Found     : BOOL;
    EmptySlot : INT;
    Executed  : BOOL := FALSE; // One-shot latch
END_VAR

// Main logic block
IF ENABLE AND NOT Executed THEN
    DONE := FALSE;
    ERROR := FALSE;
    ERR_CODE := 0;
    Found := FALSE;
    EmptySlot := 0;

    IF (COBID = 0) AND (NOT REGISTER) THEN
        // Clear all registrations
        FOR i := 1 TO 100 DO
            RegTable[i] := 0;
        END_FOR
        DONE := TRUE;
    ELSIF (COBID = 0) THEN
        ERROR := TRUE;
        ERR_CODE := 1; // Invalid COBID
    ELSE
        IF REGISTER THEN
            // Check for duplicate and find empty slot
            FOR i := 1 TO 100 DO
                IF RegTable[i] = COBID THEN
                    Found := TRUE;
                    EXIT;
                ELSIF (RegTable[i] = 0) AND (EmptySlot = 0) THEN
                    EmptySlot := i;
                END_IF
            END_FOR

            IF Found THEN
                ERROR := TRUE;
                ERR_CODE := 2; // Duplicate
            ELSIF EmptySlot > 0 THEN
                RegTable[EmptySlot] := COBID;
                DONE := TRUE;
            ELSE
                ERROR := TRUE;
                ERR_CODE := 9; // No space
            END_IF

        ELSE
            // Deregistration
            FOR i := 1 TO 100 DO
                IF RegTable[i] = COBID THEN
                    RegTable[i] := 0;
                    DONE := TRUE;
                    Found := TRUE;
                    EXIT;
                END_IF
            END_FOR

            IF NOT Found THEN
                ERROR := TRUE;
                ERR_CODE := 3; // Not found
            END_IF
        END_IF
    END_IF
    Executed := TRUE;

ELSIF NOT ENABLE THEN
    Executed := FALSE; // Reset latch
END_IF
