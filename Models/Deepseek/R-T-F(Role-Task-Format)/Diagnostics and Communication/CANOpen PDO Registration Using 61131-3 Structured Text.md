FUNCTION_BLOCK CAN_REGISTER_COBID
VAR_INPUT
    COBID : UINT;         // CAN Open Communication Object Identifier
    REGISTER : BOOL;      // TRUE to register, FALSE to deregister
END_VAR

VAR_OUTPUT
    SUCCESS : BOOL;       // Indicates successful operation
    ERROR : BOOL;         // Indicates an error occurred
END_VAR

VAR
    registeredCOBs : ARRAY[1..10] OF UINT; // Array to store registered COBIDs
    numRegistered : INT := 0;             // Number of currently registered COBs
    buffer : ARRAY[1..100] OF BYTE;        // Buffer to store incoming messages
    bufferIndex : INT := 0;               // Index for the buffer
END_VAR

// Method to clear all registrations and reset the buffer
METHOD ClearAllRegistrations : BOOL
VAR
    i : INT;
END_VAR
    FOR i := 1 TO numRegistered DO
        registeredCOBs[i] := 0;
    END_FOR;
    numRegistered := 0;
    bufferIndex := 0;
    RETURN TRUE;
END_METHOD

// Method to check if a COBID is already registered
METHOD IsCOBIDRegistered : BOOL
VAR_INPUT
    cobidToCheck : UINT;
END_VAR
VAR
    i : INT;
END_VAR
    FOR i := 1 TO numRegistered DO
        IF registeredCOBs[i] = cobidToCheck THEN
            RETURN TRUE;
        END_IF;
    END_FOR;
    RETURN FALSE;
END_METHOD

// Method to add a COBID to the registration list
METHOD RegisterCOBID : BOOL
VAR_INPUT
    cobidToAdd : UINT;
END_VAR
IF numRegistered >= SIZEOF(registeredCOBs) THEN
    RETURN FALSE; // No more space to register new COBIDs
END_IF;
registeredCOBs[numRegistered + 1] := cobidToAdd;
numRegistered := numRegistered + 1;
RETURN TRUE;
END_METHOD

// Method to remove a COBID from the registration list
METHOD DeregisterCOBID : BOOL
VAR_INPUT
    cobidToRemove : UINT;
END_VAR
VAR
    i : INT;
    found : BOOL := FALSE;
END_VAR
FOR i := 1 TO numRegistered DO
    IF registeredCOBs[i] = cobidToRemove THEN
        found := TRUE;
        BREAK;
    END_IF;
END_FOR;
IF found THEN
    FOR i := i TO numRegistered - 1 DO
        registeredCOBs[i] := registeredCOBs[i + 1];
    END_FOR;
    registeredCOBs[numRegistered] := 0;
    numRegistered := numRegistered - 1;
    RETURN TRUE;
ELSE
    RETURN FALSE;
END_IF;
END_METHOD

// Main execution logic
METHOD Execute : BOOL
IF REGISTER AND NOT IsCOBIDRegistered(COBID) THEN
    IF RegisterCOBID(COBID) THEN
        SUCCESS := TRUE;
        ERROR := FALSE;
    ELSE
        SUCCESS := FALSE;
        ERROR := TRUE;
    END_IF;
ELSIF NOT REGISTER THEN
    IF COBID = 0 THEN
        IF ClearAllRegistrations() THEN
            SUCCESS := TRUE;
            ERROR := FALSE;
        ELSE
            SUCCESS := FALSE;
            ERROR := TRUE;
        END_IF;
    ELSIF IsCOBIDRegistered(COBID) THEN
        IF DeregisterCOBID(COBID) THEN
            SUCCESS := TRUE;
            ERROR := FALSE;
        ELSE
            SUCCESS := FALSE;
            ERROR := TRUE;
        END_IF;
    ELSE
        SUCCESS := TRUE;
        ERROR := FALSE; // COBID was not registered, no action needed
    END_IF;
ELSE
    SUCCESS := TRUE;
    ERROR := FALSE; // COBID already registered, no action needed
END_IF;
RETURN SUCCESS;
END_METHOD

END_FUNCTION_BLOCK
