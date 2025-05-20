FUNCTION_BLOCK CAN_REGISTER_COBID
(*
    Function Block: CAN_REGISTER_COBID
    Purpose: Registers or deregisters a COBID for PDO/CAN Layer 2 message reception
             in a CANOpen network, with full reset capability and error handling.
    Standard: IEC 61131-3 Structured Text
*)

(* Input Variables *)
VAR_INPUT
    COBID     : DWORD;  (* CAN Object Identifier (11-bit or 29-bit) *)
    REGISTER  : BOOL;   (* TRUE: Register COBID, FALSE: Deregister COBID *)
END_VAR

(* Output Variables *)
VAR_OUTPUT
    DONE      : BOOL;   (* Operation completed successfully *)
    ERROR     : BOOL;   (* Error occurred during operation *)
    ERR_CODE  : UINT;   (* Error code: 0 = No error, 1 = Invalid COBID, 
                           2 = Buffer full, 3 = COBID not found, 4 = Network fault *)
END_VAR

(* Internal Variables *)
VAR
    (* Network buffer to store registered COBIDs *)
    COBID_Buffer : ARRAY[0..63] OF DWORD;  (* Max 64 COBIDs *)
    Buffer_Index : UINT := 0;              (* Current number of registered COBIDs *)
    
    (* Network status flags *)
    Network_Connected : BOOL := FALSE;     (* Network connection status *)
    Buffer_Initialized : BOOL := FALSE;    (* Buffer initialization status *)
    
    (* Temporary variables *)
    i : UINT;                              (* Loop counter *)
    Found : BOOL;                          (* COBID search flag *)
    Temp_COBID : DWORD;                    (* Temporary COBID storage *)
END_VAR

(* Function Block Logic *)
METHOD PRIVATE InitializeBuffer
    (* Initialize the COBID buffer *)
    FOR i := 0 TO 63 DO
        COBID_Buffer[i] := 0;
    END_FOR;
    Buffer_Index := 0;
    Buffer_Initialized := TRUE;
END_METHOD

METHOD PRIVATE CheckNetworkStatus : BOOL
    (* Simulate network status check - in practice, query CANOpen stack *)
    (* Placeholder: Assume network is connected for simulation *)
    Network_Connected := TRUE;  (* Replace with actual CANOpen status check *)
    RETURN Network_Connected;
END_METHOD

METHOD PRIVATE ValidateCOBID : BOOL
    (* Validate COBID: Ensure it’s non-zero and within CANOpen range *)
    RETURN (COBID <> 0) AND (COBID <= 16#1FFFFFFF);  (* Max 29-bit COBID *)
END_METHOD

METHOD PRIVATE FindCOBID : BOOL
    (* Search for COBID in buffer *)
    Found := FALSE;
    FOR i := 0 TO Buffer_Index - 1 DO
        IF COBID_Buffer[i] = COBID THEN
            Found := TRUE;
            EXIT;
        END_IF;
    END_FOR;
    RETURN Found;
END_METHOD

(* Main Execution Logic *)
IF NOT Buffer_Initialized THEN
    InitializeBuffer();
END_IF;

(* Reset outputs *)
DONE := FALSE;
ERROR := FALSE;
ERR_CODE := 0;

(* Check network status *)
IF NOT CheckNetworkStatus() THEN
    ERROR := TRUE;
    ERR_CODE := 4;  (* Network fault *)
    RETURN;
END_IF;

(* Handle full reset: REGISTER = FALSE and COBID = 0 *)
IF NOT REGISTER AND COBID = 0 THEN
    InitializeBuffer();
    DONE := TRUE;
    RETURN;
END_IF;

(* Validate COBID *)
IF NOT ValidateCOBID() THEN
    ERROR := TRUE;
    ERR_CODE := 1;  (* Invalid COBID *)
    RETURN;
END_IF;

IF REGISTER THEN
    (* Register COBID *)
    IF Buffer_Index >= 64 THEN
        ERROR := TRUE;
        ERR_CODE := 2;  (* Buffer full *)
        RETURN;
    END_IF;
    
    (* Check if COBID already registered *)
    IF FindCOBID() THEN
        DONE := TRUE;  (* Already registered, no action needed *)
        RETURN;
    END_IF;
    
    (* Register new COBID *)
    COBID_Buffer[Buffer_Index] := COBID;
    Buffer_Index := Buffer_Index + 1;
    DONE := TRUE;
    
ELSE
    (* Deregister COBID *)
    IF NOT FindCOBID() THEN
        ERROR := TRUE;
        ERR_CODE := 3;  (* COBID not found *)
        RETURN;
    END_IF;
    
    (* Remove COBID and shift buffer *)
    Found := FALSE;
    FOR i := 0 TO Buffer_Index - 1 DO
        IF COBID_Buffer[i] = COBID THEN
            Found := TRUE;
        END_IF;
        IF Found AND i < Buffer_Index - 1 THEN
            COBID_Buffer[i] := COBID_Buffer[i + 1];
        END_IF;
    END_FOR;
    
    IF Found THEN
        COBID_Buffer[Buffer_Index - 1] := 0;
        Buffer_Index := Buffer_Index - 1;
        DONE := TRUE;
    ELSE
        ERROR := TRUE;
        ERR_CODE := 3;  (* COBID not found *)
    END_IF;
END_IF;

(* Simulate CANOpen stack interaction *)
(* In practice, call CANOpen API to register/deregister COBID *)
(* e.g., CANOpen_SetRxFilter(COBID, REGISTER); *)
END_FUNCTION_BLOCK
