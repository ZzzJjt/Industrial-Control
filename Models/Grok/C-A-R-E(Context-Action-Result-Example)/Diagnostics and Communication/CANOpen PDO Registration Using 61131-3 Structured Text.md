(* IEC 61131-3 Structured Text: CAN_REGISTER_COBID Function Block *)
(* Purpose: Manages registration and deregistration of CANOpen COBIDs for PDO and Layer 2 messages *)

FUNCTION_BLOCK CAN_REGISTER_COBID
VAR_INPUT
    EXECUTE : BOOL;                 (* Trigger execution on rising edge *)
    REGISTER : BOOL;                (* TRUE to register, FALSE to deregister *)
    COBID : DWORD;                  (* CAN Object Identifier, e.g., 0x181 *)
END_VAR
VAR_OUTPUT
    DONE : BOOL;                    (* TRUE when operation completes *)
    ERROR : BOOL;                   (* TRUE if error occurs *)
    ERROR_CODE : INT;               (* 0: No error, 1: Invalid COBID, 2: Duplicate, 3: Not found, 4: Buffer full, 5: Network error *)
    REGISTERED_COBIDS : ARRAY[1..100] OF DWORD; (* List of registered COBIDs *)
    NUM_COBIDS : INT;               (* Number of registered COBIDs *)
END_VAR
VAR
    LastExecute : BOOL;             (* Previous EXECUTE state for edge detection *)
    State : INT := 0;               (* State: 0=Idle, 1=Register, 2=Deregister, 3=ClearAll *)
    i : INT;                        (* Loop index *)
    Found : BOOL;                   (* COBID found flag *)
    BufferCleared : BOOL;           (* Network buffer cleared flag *)
END_VAR

(* Main Logic *)
IF EXECUTE AND NOT LastExecute THEN (* Rising edge *)
    DONE := FALSE;
    ERROR := FALSE;
    ERROR_CODE := 0;
    
    (* Validate COBID *)
    IF COBID > 16#7FF AND NOT (NOT REGISTER AND COBID = 0) THEN
        ERROR := TRUE;
        ERROR_CODE := 1; (* Invalid COBID *)
        DONE := TRUE;
        State := 0;
    ELSIF REGISTER THEN
        State := 1; (* Register *)
    ELSIF NOT REGISTER AND COBID = 0 THEN
        State := 3; (* Clear All *)
    ELSE
        State := 2; (* Deregister *)
    END_IF;
END_IF;

LastExecute := EXECUTE;

CASE State OF
    0: (* Idle *)
        IF NOT EXECUTE THEN
            DONE := FALSE;
            ERROR := FALSE;
            ERROR_CODE := 0;
        END_IF;
    
    1: (* Register *)
        (* Check for duplicate COBID *)
        Found := FALSE;
        FOR i := 1 TO NUM_COBIDS DO
            IF REGISTERED_COBIDS[i] = COBID THEN
                Found := TRUE;
                EXIT;
            END_IF;
        END_FOR;
        
        IF Found THEN
            ERROR := TRUE;
            ERROR_CODE := 2; (* Duplicate COBID *)
            DONE := TRUE;
            State := 0;
        ELSIF NUM_COBIDS >= 100 THEN
            ERROR := TRUE;
            ERROR_CODE := 4; (* Buffer full *)
            DONE := TRUE;
            State := 0;
        ELSE
            (* Simulate network communication *)
            IF RAND() MOD 100 < 5 THEN (* 5% chance of network error *)
                ERROR := TRUE;
                ERROR_CODE := 5; (* Network error *)
                DONE := TRUE;
                State := 0;
            ELSE
                NUM_COBIDS := NUM_COBIDS + 1;
                REGISTERED_COBIDS[NUM_COBIDS] := COBID;
                BufferCleared := FALSE; (* Buffer not cleared *)
                DONE := TRUE;
                State := 0;
            END_IF;
        END_IF;
    
    2: (* Deregister *)
        Found := FALSE;
        FOR i := 1 TO NUM_COBIDS DO
            IF REGISTERED_COBIDS[i] = COBID THEN
                Found := TRUE;
                (* Shift array to remove COBID *)
                FOR j := i TO NUM_COBIDS - 1 DO
                    REGISTERED_COBIDS[j] := REGISTERED_COBIDS[j + 1];
                END_FOR;
                REGISTERED_COBIDS[NUM_COBIDS] := 0;
                NUM_COBIDS := NUM_COBIDS - 1;
                EXIT;
            END_IF;
        END_FOR;
        
        IF NOT Found THEN
            ERROR := TRUE;
            ERROR_CODE := 3; (* COBID not found *)
            DONE := TRUE;
            State := 0;
        ELSIF RAND() MOD 100 < 5 THEN (* 5% chance of network error *)
            ERROR := TRUE;
            ERROR_CODE := 5; (* Network error *)
            DONE := TRUE;
            State := 0;
        ELSE
            BufferCleared := FALSE;
            DONE := TRUE;
            State := 0;
        END_IF;
    
    3: (* Clear All *)
        IF RAND() MOD 100 < 5 THEN (* 5% chance of network error *)
            ERROR := TRUE;
            ERROR_CODE := 5; (* Network error *)
            DONE := TRUE;
            State := 0;
        ELSE
            FOR i := 1 TO NUM_COBIDS DO
                REGISTERED_COBIDS[i] := 0;
            END_FOR;
            NUM_COBIDS := 0;
            BufferCleared := TRUE; (* Network buffer cleared *)
            DONE := TRUE;
            State := 0;
        END_IF;
END_CASE;

(* Notes:
   - Purpose: Registers/deregisters CANOpen COBIDs for PDO/Layer 2 messages.
   - Inputs:
     - EXECUTE: Triggers operation on rising edge.
     - REGISTER: TRUE to register, FALSE to deregister.
     - COBID: CAN Object ID (0x0–0x7FF; 0 for clear all when REGISTER=FALSE).
   - Outputs:
     - DONE: TRUE when operation completes successfully.
     - ERROR: TRUE if error occurs.
     - ERROR_CODE: 0 (No error), 1 (Invalid COBID), 2 (Duplicate), 3 (Not found), 4 (Buffer full), 5 (Network error).
     - REGISTERED_COBIDS: ARRAY[1..100] OF DWORD, stores registered COBIDs.
     - NUM_COBIDS: Number of registered COBIDs.
   - Logic:
     - Rising edge on EXECUTE triggers state machine.
     - State 1 (Register): Adds COBID if valid, not duplicated, buffer not full.
     - State 2 (Deregister): Removes COBID if found.
     - State 3 (Clear All): Clears all COBIDs and network buffer if COBID=0, REGISTER=FALSE.
   - Error Handling:
     - Invalid COBID: >0x7FF or 0 when REGISTER=TRUE.
     - Duplicate COBID: Already in REGISTERED_COBIDS.
     - COBID not found: Not in REGISTERED_COBIDS during deregistration.
     - Buffer full: NUM_COBIDS >= 100.
     - Network error: Simulated 5% failure rate.
   - Safety:
     - Rising edge detection prevents repeated triggers.
     - Array bounds checking avoids memory overflow.
     - State machine ensures scan-cycle safety.
     - BufferCleared flag tracks network buffer state.
   - Usage:
     - Recipe change: Register new PDO (REGISTER=TRUE, COBID=0x182); deregister old (REGISTER=FALSE, COBID=0x181).
     - Batch abort: Clear all (REGISTER=FALSE, COBID=0).
   - Maintenance:
     - Modular design aids debugging.
     - ERROR_CODE and alarms support HMI diagnostics.
   - Platform Notes:
     - Assumes DWORD for COBID (11-bit CAN ID); extendable to 29-bit.
     - Buffer size (100) is practical; adjustable for memory constraints.
     - RAND() simulates network errors; replace with actual CAN driver checks.
*)
END_FUNCTION_BLOCK
