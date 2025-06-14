FUNCTION_BLOCK CAN_REGISTER_COBID
VAR_INPUT
    REGISTER : BOOL;
    COBID    : DWORD;
END_VAR

VAR_OUTPUT
    SUCCESS     : BOOL;
    ERROR_CODE  : INT;
END_VAR

VAR
    registeredList : ARRAY[1..10] OF DWORD := [0,0,0,0,0,0,0,0,0,0];
    i              : INT;
    isRegistered   : BOOL;
END_VAR

(* Reset status flags *)
SUCCESS := FALSE;
ERROR_CODE := 0;

(* Check communication availability (pseudo-check) *)
IF NOT ComStatus_OK() THEN
    ERROR_CODE := -1; // Communication not available
    RETURN;
END_IF

(* Clear all registrations if COBID = 0 and REGISTER = FALSE *)
IF NOT REGISTER AND (COBID = 0) THEN
    FOR i := 1 TO 10 DO
        registeredList[i] := 0;
    END_FOR
    ClearNetworkBuffer();
    SUCCESS := TRUE;
    RETURN;
END_IF

(* Check if COBID is already registered *)
isRegistered := FALSE;
FOR i := 1 TO 10 DO
    IF registeredList[i] = COBID THEN
        isRegistered := TRUE;
        EXIT;
    END_IF
END_FOR

(* REGISTER = TRUE => Add COBID to list if not already registered *)
IF REGISTER THEN
    IF NOT isRegistered THEN
        FOR i := 1 TO 10 DO
            IF registeredList[i] = 0 THEN
                registeredList[i] := COBID;
                SUCCESS := TRUE;
                RETURN;
            END_IF
        END_FOR
        ERROR_CODE := -2; // No space left
    ELSE
        SUCCESS := TRUE; // Already registered
    END_IF
ELSE
    (* REGISTER = FALSE => Deregister COBID *)
    IF isRegistered THEN
        FOR i := 1 TO 10 DO
            IF registeredList[i] = COBID THEN
                registeredList[i] := 0;
                SUCCESS := TRUE;
                RETURN;
            END_IF
        END_FOR
    ELSE
        ERROR_CODE := -3; // Not registered
    END_IF
END_IF
