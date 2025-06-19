(* IEC 61131-3 Structured Text: Heapsort Function Block *)
(* Purpose: Sorts an array of integers using iterative heapsort algorithm *)

FUNCTION_BLOCK HeapSort
VAR_INPUT
    Execute : BOOL;                 (* TRUE to initiate sorting *)
    InputArray : ARRAY[1..100] OF INT; (* Input array to sort *)
    ArraySize : INT;                (* Number of elements to sort, 1 to 100 *)
END_VAR
VAR_OUTPUT
    SortedArray : ARRAY[1..100] OF INT; (* Sorted output array *)
    Done : BOOL;                    (* TRUE when sorting is complete *)
    Error : BOOL;                   (* TRUE if input validation fails *)
END_VAR
VAR
    HeapSize : INT;                 (* Current size of the heap *)
    i : INT;                        (* Loop counter *)
    Temp : INT;                     (* Temporary variable for swapping *)
    Parent : INT;                   (* Parent index *)
    Child : INT;                    (* Child index *)
    RootValue : INT;                (* Root value for heapify *)
    Valid : BOOL;                   (* Input validation flag *)
END_VAR

(* Iterative Max-Heapify Procedure *)
METHOD PRIVATE Heapify
VAR_INPUT
    Root : INT;                     (* Index of root node to heapify *)
END_VAR
(* Ensures max-heap property: parent >= children *)
Parent := Root;
RootValue := SortedArray[Parent];

WHILE TRUE DO
    (* Find largest among root and children *)
    Child := 2 * Parent;  (* Left child *)
    IF Child <= HeapSize THEN
        IF Child < HeapSize AND SortedArray[Child] < SortedArray[Child + 1] THEN
            Child := Child + 1;  (* Right child is larger *)
        END_IF;
        IF RootValue < SortedArray[Child] THEN
            (* Move child up *)
            SortedArray[Parent] := SortedArray[Child];
            Parent := Child;
        ELSE
            (* Heap property satisfied *)
            EXIT;
        END_IF;
    ELSE
        (* No children, exit *)
        EXIT;
    END_IF;
END_WHILE;

(* Restore root value *)
SortedArray[Parent] := RootValue;
END_METHOD

(* Main Logic *)
IF NOT Execute THEN
    (* Reset outputs when not executing *)
    Done := FALSE;
    Error := FALSE;
    FOR i := 1 TO ArraySize DO
        SortedArray[i] := InputArray[i];  (* Copy input to output *)
    END_FOR;
ELSE
    (* Input Validation *)
    IF ArraySize < 1 OR ArraySize > 100 THEN
        (* Invalid array size *)
        Error := TRUE;
        Done := FALSE;
        SortedArray := InputArray;  (* Return unchanged array *)
    ELSE
        Error := FALSE;
        Done := FALSE;
        
        (* Copy InputArray to SortedArray for in-place sorting *)
        FOR i := 1 TO ArraySize DO
            SortedArray[i] := InputArray[i];
        END_FOR;
        
        (* Step 1: Build Max Heap *)
        HeapSize := ArraySize;
        FOR i := HeapSize / 2 TO 1 BY -1 DO
            (* Start from last non-leaf node, heapify backwards *)
            THIS.Heapify(i);
        END_FOR;
        
        (* Step 2: Extract Elements from Heap *)
        WHILE HeapSize > 1 DO
            (* Swap root (max) with last element *)
            Temp := SortedArray[1];
            SortedArray[1] := SortedArray[HeapSize];
            SortedArray[HeapSize] := Temp;
            
            (* Reduce heap size and heapify root *)
            HeapSize := HeapSize - 1;
            THIS.Heapify(1);
        END_WHILE;
        
        (* Sorting complete *)
        Done := TRUE;
    END_IF;
END_IF;

(* Notes:
   - Purpose: Sorts an array of integers using iterative heapsort for PLC environments.
   - Inputs:
     - Execute: Initiates sorting when TRUE.
     - InputArray: ARRAY[1..100] OF INT, input array to sort.
     - ArraySize: Number of elements to sort (1 to 100).
   - Outputs:
     - SortedArray: Sorted array (ascending order).
     - Done: TRUE when sorting is complete.
     - Error: TRUE if ArraySize is invalid (<1 or >100).
   - Algorithm:
     - Heapsort: O(n log n) complexity, in-place sorting.
     - Builds max heap iteratively, starting from last non-leaf node.
     - Extracts max element, reduces heap size, and restores heap property.
   - Features:
     - Iterative: Avoids recursion for PLC compatibility.
     - No DOWNTO: Uses ascending FOR loops with index manipulation.
     - In-Place: Sorts within SortedArray to minimize memory usage.
     - Single-cycle execution for ArraySize <= 100; multi-cycle logic can be added.
   - Safety:
     - Validates ArraySize to prevent array access errors.
     - Resets outputs when Execute = FALSE.
     - Iterative Heapify prevents stack overflow.
   - Usage:
     - Ideal for prioritization, scheduling, or sensor data organization in automation.
     - Reusable in IEC 61131-3 programs (e.g., CODESYS, TwinCAT).
   - Maintenance:
     - Modular design with clear comments aids debugging.
     - Error and Done flags support diagnostics.
   - Platform Notes:
     - Assumes INT array; can be adapted for DINT or other types.
     - ArraySize <= 100 balances flexibility and PLC memory constraints.
*)
END_FUNCTION_BLOCK
