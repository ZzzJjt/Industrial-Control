(* IEC 61131-3 Structured Text Function Block for Binary Search *)
(* Searches for a target value in a sorted 100-element integer array *)
(* Conforms to industrial automation best practices *)

FUNCTION_BLOCK BinarySearch
VAR_INPUT
    SearchArray: ARRAY[1..100] OF INT; (* Sorted input array, ascending order *)
    TargetValue: INT;                  (* Value to search for *)
    Enable: BOOL;                      (* Trigger to start search *)
END_VAR

VAR_OUTPUT
    Index: INT;                        (* Index of target value, 0 if not found *)
    Found: BOOL;                       (* TRUE if target value is found *)
END_VAR

VAR
    Low: INT;                          (* Lower bound of search window *)
    High: INT;                         (* Upper bound of search window *)
    Mid: INT;                          (* Midpoint of search window *)
    ValidInput: BOOL;                  (* Input validation flag *)
    SearchComplete: BOOL;              (* Flag to indicate search completion *)
END_VAR

(* Main Function Block Logic *)
(* Reset outputs when not enabled *)
IF NOT Enable THEN
    Index := 0;
    Found := FALSE;
    SearchComplete := FALSE;
    ValidInput := FALSE;
    Low := 0;
    High := 0;
    Mid := 0;
    RETURN;
END_IF;

(* Step 1: Input Validation *)
(* Ensure Enable is TRUE and array bounds are valid *)
ValidInput := Enable;

(* Step 2: Initialize Search *)
IF ValidInput AND NOT SearchComplete THEN
    Low := 1;                      (* Start at first array element *)
    High := 100;                   (* End at last array element *)
    Index := 0;                    (* Reset index *)
    Found := FALSE;                (* Reset found flag *)
END_IF;

(* Step 3: Binary Search Algorithm *)
WHILE ValidInput AND NOT SearchComplete AND Low <= High DO
    (* Calculate midpoint, avoiding integer overflow *)
    Mid := Low + (High - Low) / 2;
    
    (* Compare target with middle element *)
    IF SearchArray[Mid] = TargetValue THEN
        (* Target found *)
        Index := Mid;
        Found := TRUE;
        SearchComplete := TRUE;
    ELSIF SearchArray[Mid] > TargetValue THEN
        (* Target is in lower half *)
        High := Mid - 1;
    ELSE
        (* Target is in upper half *)
        Low := Mid + 1;
    END_IF;
    
    (* Check if search window is exhausted *)
    IF Low > High THEN
        SearchComplete := TRUE;
        Index := 0;
        Found := FALSE;
    END_IF;
END_WHILE;

(* Step 4: Finalize Outputs *)
(* Ensure outputs are set correctly after search *)
IF SearchComplete THEN
    (* Outputs already set in loop *)
ELSE
    (* In case of early termination or invalid input *)
    Index := 0;
    Found := FALSE;
END_IF;

(* Step 5: Safety Check *)
(* Ensure array access is within bounds *)
IF Mid < 1 OR Mid > 100 THEN
    Index := 0;
    Found := FALSE;
    SearchComplete := TRUE;
END_IF;

END_FUNCTION_BLOCK
