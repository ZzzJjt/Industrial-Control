(* Validate COBID: Must be in valid CANOpen range (1 to 0x7FF) or 0 for reset *)
IF COBID > 16#7FF AND COBID <> 0 THEN
    ERROR := TRUE;
    ERROR_CODE := 1; (* Invalid COBID *)
    BUSY := FALSE;
    DONE := FALSE;
ELSIF REGISTER THEN
    (* Register COBID *)
    (* Check for duplicate registration *)
    Found := FALSE;
    FOR i := 0 TO NumRegistered - 1 DO
        IF RegisteredCOBIDs[i] = COBID THEN
            Found := TRUE;
            EXIT;
        END_IF
    END_FOR;
    
    IF Found THEN
        ERROR := TRUE;
        ERROR_CODE := 2; (* Duplicate COBID *)
        BUSY := FALSE;
        DONE := FALSE;
    ELSIF NumRegistered >= 100 THEN
        ERROR := TRUE;
        ERROR_CODE := 4; (* Buffer Full *)
        BUSY := FALSE;
        DONE := FALSE;
    ELSE
        (* Simulate network registration *)
        (* Assume CAN_NetworkRegister(COBID) returns 0 on success, -1 on failure *)
        IF CAN_NetworkRegister(COBID) = 0 THEN
            RegisteredCOBIDs[NumRegistered] := COBID;
            NumRegistered := NumRegistered + 1;
            DONE := TRUE;
        ELSE
            ERROR := TRUE;
            ERROR_CODE := 3; (* Network Error *)
            DONE := FALSE;
        END_IF
        BUSY := FALSE;
    END_IF
ELSE (* REGISTER = FALSE *)
    IF COBID = 0 THEN
        (* Clear all COBIDs and purge network buffer *)
        FOR i := 0 TO NumRegistered - 1 DO
            IF CAN_NetworkDeregister(RegisteredCOBIDs[i]) <> 0 THEN
                ERROR := TRUE;
                ERROR_CODE := 3; (* Network Error *)
                BUSY := FALSE;
                DONE := FALSE;
                RETURN;
            END_IF
        END_FOR;
        NumRegistered := 0;
        BufferCleared := CAN_PurgeNetworkBuffer(); (* Simulate buffer purge *)
        IF NOT BufferCleared THEN
            ERROR := TRUE;
            ERROR_CODE := 3; (* Network Error *)
        ELSE
            DONE := TRUE;
        END_IF
        BUSY := FALSE;
    ELSE
        (* Deregister specific COBID *)
        Found := FALSE;
        FOR i := 0 TO NumRegistered - 1 DO
            IF RegisteredCOBIDs[i] = COBID THEN
                Found := TRUE;
                EXIT;
            END_IF
        END_FOR;
        
        IF NOT Found THEN
            ERROR := TRUE;
            ERROR_CODE := 1; (* Invalid COBID *)
            BUSY := FALSE;
            DONE := FALSE;
        ELSE
            IF CAN_NetworkDeregister(COBID) = 0 THEN
                (* Shift array to remove COBID *)
                FOR i := i TO NumRegistered - 2 DO
                    RegisteredCOBIDs[i] := RegisteredCOBIDs[i + 1];
                END_FOR;
                NumRegistered := NumRegistered - 1;
                DONE := TRUE;
            ELSE
                ERROR := TRUE;
                ERROR_CODE := 3; (* Network Error *)
                DONE := FALSE;
            END_IF
            BUSY := FALSE;
        END_IF
    END_IF
END_IF
