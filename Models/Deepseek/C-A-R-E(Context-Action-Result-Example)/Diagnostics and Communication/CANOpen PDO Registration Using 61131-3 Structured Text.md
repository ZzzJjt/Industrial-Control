FUNCTION_BLOCK CAN_REGISTER_COBID
VAR_INPUT
    REGISTER : BOOL; // TRUE to register, FALSE to deregister
    COBID : UDINT;   // CAN Open ID to register or deregister
END_VAR

VAR_OUTPUT
    STATUS : INT;    // Status code: 0 = OK, -1 = Invalid COBID, -2 = Duplicate Registration, -3 = Deregistration Error, -4 = Buffer Clear Error
END_VAR

VAR
    registeredCOBIDs : ARRAY[1..100] OF UDINT; // Array to store registered COBIDs
    numRegistered : INT := 0;                  // Number of currently registered COBIDs
END_VAR

METHOD RegisterCOBID : INT
VAR_INPUT
    cobid : UDINT;
END_VAR
VAR
    i : INT;
END_VAR
    FOR i := 1 TO numRegistered DO
        IF registeredCOBIDs[i] = cobid THEN
            RETURN -2; // Duplicate Registration
        END_IF
    END_FOR
    IF numRegistered >= SIZEOF(registeredCOBIDs, 1) THEN
        RETURN -1; // No space left for new registration
    END_IF
    numRegistered := numRegistered + 1;
    registeredCOBIDs[numRegistered] := cobid;
    RETURN 0; // Success
END_METHOD

METHOD DeregisterCOBID : INT
VAR_INPUT
    cobid : UDINT;
END_VAR
VAR
    i, j : INT;
    found : BOOL := FALSE;
END_VAR
    FOR i := 1 TO numRegistered DO
        IF registeredCOBIDs[i] = cobid THEN
            found := TRUE;
            FOR j := i TO numRegistered - 1 DO
                registeredCOBIDs[j] := registeredCOBIDs[j + 1];
            END_FOR
            numRegistered := numRegistered - 1;
            EXIT;
        END_IF
    END_FOR
    IF NOT found THEN
        RETURN -3; // Deregistration Error: COBID not found
    END_IF
    RETURN 0; // Success
END_METHOD

METHOD ClearBuffer : INT
VAR
    i : INT;
END_VAR
    FOR i := 1 TO numRegistered DO
        registeredCOBIDs[i] := 0;
    END_FOR
    numRegistered := 0;
    RETURN 0; // Success
END_METHOD

// Main execution logic
IF REGISTER THEN
    IF COBID = 0 THEN
        STATUS := -1; // Invalid COBID
    ELSE
        STATUS := RegisterCOBID(COBID);
    END_IF
ELSE
    IF COBID = 0 THEN
        STATUS := ClearBuffer();
    ELSE
        STATUS := DeregisterCOBID(COBID);
    END_IF
END_IF



