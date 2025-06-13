FUNCTION_BLOCK CAN_REGISTER_COBID
VAR_INPUT
    REGISTER : BOOL; // TRUE to register, FALSE to deregister
    COBID : UINT;   // CAN Open COB ID to register/deregister
END_VAR

VAR_OUTPUT
    STATUS : INT;   // Status code: 0 = OK, -1 = Error
    MESSAGE : STRING[100]; // Detailed message about the status
END_VAR

VAR
    registeredCOBs : ARRAY[1..10] OF UINT; // Array to store registered COB IDs
    numRegistered : INT := 0; // Number of currently registered COBs
    maxRegistrations : INT := 10; // Maximum number of registrations
    networkBuffer : ARRAY[1..10] OF BYTEARRAY[8]; // Network buffer for received messages
END_VAR

METHOD RegisterCOB : BOOL
VAR_INPUT
    cobid : UINT;
END_VAR
VAR
    i : INT;
    found : BOOL := FALSE;
BEGIN
    IF numRegistered >= maxRegistrations THEN
        STATUS := -1;
        MESSAGE := 'Maximum number of registrations reached.';
        RETURN FALSE;
    END_IF;

    FOR i := 1 TO numRegistered DO
        IF registeredCOBs[i] = cobid THEN
            found := TRUE;
            EXIT;
        END_IF;
    END_FOR;

    IF NOT found THEN
        numRegistered := numRegistered + 1;
        registeredCOBs[numRegistered] := cobid;
        // Simulate registering with the CAN driver
        // This would typically involve calling a low-level CAN API
        STATUS := 0;
        MESSAGE := 'Registration successful.';
        RETURN TRUE;
    ELSE
        STATUS := -1;
        MESSAGE := 'COB ID already registered.';
        RETURN FALSE;
    END_IF;
END_METHOD

METHOD DeregisterCOB : BOOL
VAR_INPUT
    cobid : UINT;
END_VAR
VAR
    i, j : INT;
    found : BOOL := FALSE;
BEGIN
    FOR i := 1 TO numRegistered DO
        IF registeredCOBs[i] = cobid THEN
            found := TRUE;
            FOR j := i TO numRegistered - 1 DO
                registeredCOBs[j] := registeredCOBs[j + 1];
                networkBuffer[j] := networkBuffer[j + 1];
            END_FOR;
            numRegistered := numRegistered - 1;
            // Simulate deregistering with the CAN driver
            // This would typically involve calling a low-level CAN API
            STATUS := 0;
            MESSAGE := 'Deregistration successful.';
            RETURN TRUE;
        END_IF;
    END_FOR;

    IF NOT found THEN
        STATUS := -1;
        MESSAGE := 'COB ID not found.';
        RETURN FALSE;
    END_IF;
END_METHOD

METHOD ClearAllRegistrations : BOOL
BEGIN
    numRegistered := 0;
    // Simulate clearing all registrations with the CAN driver
    // This would typically involve calling a low-level CAN API
    STATUS := 0;
    MESSAGE := 'All registrations cleared.';
    RETURN TRUE;
END_METHOD

METHOD Execute : BOOL
BEGIN
    IF REGISTER AND (COBID <> 0) THEN
        RETURN RegisterCOB(COBID);
    ELSIF NOT REGISTER AND (COBID <> 0) THEN
        RETURN DeregisterCOB(COBID);
    ELSIF NOT REGISTER AND (COBID = 0) THEN
        RETURN ClearAllRegistrations();
    ELSE
        STATUS := -1;
        MESSAGE := 'Invalid input combination.';
        RETURN FALSE;
    END_IF;
END_METHOD

END_FUNCTION_BLOCK



