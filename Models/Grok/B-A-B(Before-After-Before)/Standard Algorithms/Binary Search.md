(* IEC 61131-3 Structured Text: Binary Search Function Block *)
(* Purpose: Efficiently searches a sorted 100-element integer array using binary search *)

FUNCTION_BLOCK BinarySearch
VAR_INPUT
    Enable : BOOL;                  (* TRUE to initiate search *)
    SearchArray : ARRAY[1..100] OF INT; (* Sorted array of 100 integers *)
    TargetValue : INT;              (* Value to search for *)
END_VAR
VAR_OUTPUT
    Found : BOOL;                   (* TRUE if TargetValue is found *)
    Index : INT;                    (* Index of TargetValue if found, else -1 *)
    Error : BOOL;                   (* TRUE if input validation fails *)
END_VAR
VAR
    Low : INT := 1;                 (* Lower search boundary *)
    High : INT := 100;              (* Upper search boundary *)
    Mid : INT;                      (* Middle index for binary search *)
    IsSorted : BOOL := TRUE;        (* Flag to verify array is sorted *)
    i : INT;                        (* Loop counter for validation *)
END_VAR

(* Main Logic *)
IF NOT Enable THEN
    (* Reset outputs when disabled *)
    Found := FALSE;
    Index := -1;
    Error := FALSE;
    Low := 1;
    High := 100;
ELSE
    (* Input Validation *)
    (* Check if array is sorted in ascending order *)
    IsSorted := TRUE;
    FOR i := 1 TO 99 DO
        IF SearchArray[i] > SearchArray[i + 1] THEN
            IsSorted := FALSE;
            EXIT;
        END_IF;
    END_FOR;
    
    IF NOT IsSorted THEN
        (* Array is not sorted, return error *)
        Error := TRUE;
        Found := FALSE;
        Index := -1;
    ELSE
        (* Initialize search *)
        Error := FALSE;
        Found := FALSE;
        Index := -1;
        Low := 1;
        High := 100;
        
        (* Binary Search Logic *)
        WHILE Low <= High AND NOT Found DO
            Mid := (Low + High) / 2;  (* Calculate middle index *)
            
            IF SearchArray[Mid] = TargetValue THEN
                (* Target found *)
                Found := TRUE;
                Index := Mid;
            ELSIF SearchArray[Mid] > TargetValue THEN
                (* Target is in lower half *)
                High := Mid - 1;
            ELSE
                (* Target is in upper half *)
                Low := Mid + 1;
            END_IF;
        END_WHILE;
    END_IF;
END_IF;

(* Notes:
   - Purpose: Performs binary search on a sorted 100-element integer array for efficient lookup.
   - Inputs:
     - Enable: Activates the search when TRUE.
     - SearchArray: ARRAY[1..100] OF INT, must be sorted in ascending order.
     - TargetValue: Integer value to find.
   - Outputs:
     - Found: TRUE if TargetValue is found in SearchArray.
     - Index: Position (1..100) of TargetValue if found, else -1.
     - Error: TRUE if SearchArray is not sorted.
   - Validation:
     - Checks if SearchArray is sorted (ascending order).
     - Ensures index boundaries (Low, High, Mid) stay within 1..100.
   - Algorithm:
     - Binary search divides search space in half each iteration.
     - Complexity: O(log n), efficient for 100 elements (max ~7 iterations).
   - Safety:
     - Prevents infinite loops by checking Low <= High.
     - Resets outputs when Enable is FALSE.
   - Usage:
     - Ideal for time-sensitive PLC applications (e.g., sorting, threshold detection).
     - Reusable in any IEC 61131-3 program requiring fast array lookup.
   - Maintenance:
     - Clear comments and modular design aid debugging.
     - Error flag supports diagnostic logging.
*)
END_FUNCTION_BLOCK
