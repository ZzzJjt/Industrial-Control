(* IEC 61131-3 Structured Text function block for binary search *)
(* Searches a sorted array of 100 integers for a target value *)
(* Optimized for modularity and real-time PLC execution *)

FUNCTION_BLOCK BinarySearch
VAR_INPUT
    Enable : BOOL; (* Enable search execution *)
    Target : INT; (* Target value to find *)
    Array : ARRAY[0..99] OF INT; (* Sorted array of 100 integers *)
END_VAR

VAR_OUTPUT
    Index : INT := -1; (* Index of found value, -1 if not found *)
    Found : BOOL := FALSE; (* TRUE if target is found *)
    Fault : BOOL := FALSE; (* TRUE if input validation fails *)
END_VAR

VAR
    Low : INT := 0; (* Lower search boundary *)
    High : INT := 99; (* Upper search boundary *)
    Mid : INT; (* Middle index for binary search *)
    SearchActive : BOOL := FALSE; (* Flag to control search loop *)
END_VAR

(* Reset outputs on disable *)
IF NOT Enable THEN
    Index := -1;
    Found := FALSE;
    Fault := FALSE;
    SearchActive := FALSE;
    RETURN;
END_IF;

(* Input validation *)
IF Enable THEN
    (* Check array bounds *)
    IF High < 0 OR High > 99 OR Low < 0 OR Low > High THEN
        Fault := TRUE;
        Index := -1;
        Found := FALSE;
        RETURN;
    END_IF;
    
    (* Initialize search *)
    Low := 0;
    High := 99;
    SearchActive := TRUE;
    Index := -1;
    Found := FALSE;
    Fault := FALSE;
END_IF;

(* Binary search logic *)
WHILE SearchActive AND NOT Fault DO
    (* Check if search range is valid *)
    IF Low > High THEN
        SearchActive := FALSE; (* Target not found *)
        Index := -1;
        Found := FALSE;
        EXIT;
    END_IF;
    
    (* Calculate middle index *)
    Mid := Low + (High - Low) / 2; (* Avoids integer overflow *)
    
    (* Compare target with middle element *)
    IF Array[Mid] = Target THEN
        Index := Mid; (* Target found *)
        Found := TRUE;
        SearchActive := FALSE;
    ELSIF Array[Mid] > Target THEN
        High := Mid - 1; (* Search lower half *)
    ELSE
        Low := Mid + 1; (* Search upper half *)
    END_IF;
END_WHILE;

END_FUNCTION_BLOCK

(* Performance Notes *)
(* - The binary search requires at most 7 iterations for 100 elements (log2(100) ≈ 6.64), ensuring low scan cycle impact. *)
(* - The WHILE loop is bounded by the search range, preventing runaway execution in PLC environments. *)
(* - Input validation ensures robustness, but array sorting must be guaranteed externally. *)
(* - For scan-cycle-sensitive systems, the function block executes in a single cycle unless interrupted, minimizing latency. *)
