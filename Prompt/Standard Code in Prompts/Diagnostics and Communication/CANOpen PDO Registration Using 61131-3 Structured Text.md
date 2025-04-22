```
FUNCTION_BLOCK CAN_REGISTER_COBID
VAR_INPUT
    REGISTER : BOOL; (* TRUE to register, FALSE to unregister *)
    COBID : USINT; (* CAN identifier *)
VAR_OUTPUT
    REGISTERED : BOOL;
    ERROR : BOOL;
VAR
    BUFFER : ARRAY [0..100] OF BYTE := {100{0}}; (* Example buffer size *)
    BUFFER_SIZE : USINT := 0;
    BUFFER_INDEX : USINT := 0;
    REGISTERED_IDS : ARRAY [0..100] OF USINT := {100{0}}; (* Track registered COBIDs *)
    REGISTERED_COUNT : USINT := 0;
END_VAR

(* Register COBID *)
IF REGISTER THEN
    IF COBID <> 0 THEN
        (* Check if COBID is already registered *)
        FOR i := 0 TO REGISTERED_COUNT LOOP
            IF REGISTERED_IDS[i] = COBID THEN
                REGISTERED := TRUE;
                EXIT;
            END_IF
        END_FOR
        
        (* Register new COBID *)
        IF REGISTERED_COUNT < 100 THEN
            REGISTERED_IDS[REGISTERED_COUNT] := COBID;
            REGISTERED_COUNT := REGISTERED_COUNT + 1;
            REGISTERED := TRUE;
        ELSE
            ERROR := TRUE;
        END_IF
    ELSE
        ERROR := TRUE; (* Invalid COBID value *)
    END_IF
ELSIF NOT REGISTER THEN
    IF COBID = 0 THEN
        (* Clear all registered COBIDs and buffer *)
        REGISTERED_COUNT := 0;
        BUFFER_INDEX := 0;
        BUFFER_SIZE := 0;
        BUFFER := {100{0}};
        REGISTERED := TRUE;
    ELSE
        (* Unregister COBID *)
        FOR i := 0 TO REGISTERED_COUNT LOOP
            IF REGISTERED_IDS[i] = COBID THEN
                REGISTERED_IDS[i] := 0;
                REGISTERED := TRUE;
                EXIT;
            END_IF
        END_FOR
    END_IF
END_IF
END_FUNCTION_BLOCK
```
