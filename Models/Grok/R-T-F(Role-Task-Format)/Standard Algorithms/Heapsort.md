(* IEC 61131-3 Structured Text function block for heapsort algorithm *)
(* Sorts an array of integers in ascending order using iterative max heap *)
(* Optimized for PLC scan-cycle execution, in-place sorting, and ascending indices *)

FUNCTION_BLOCK HeapSort
VAR_INPUT
    Execute : BOOL; (* TRUE to start sorting *)
    InputArray : ARRAY[1..100] OF INT; (* Array to sort *)
    ArraySize : INT := 100; (* Number of elements to sort, 1 to 100 *)
END_VAR

VAR_OUTPUT
    SortedArray : ARRAY[1..100] OF INT; (* Sorted array, same as InputArray *)
    Done : BOOL := FALSE; (* TRUE when sorting is complete *)
    Fault : BOOL := FALSE; (* TRUE if invalid inputs detected *)
END_VAR

VAR
    HeapSize : INT; (* Current size of the heap *)
    i : INT; (* Loop variable for heap construction *)
    j : INT; (* Loop variable for sorting phase *)
    Temp : INT; (* Temporary variable for swapping *)
    Parent : INT; (* Parent node index *)
    Child : INT; (* Child node index *)
    Largest : INT; (* Index of largest value in heapify *)
    Sorting : BOOL := FALSE; (* Flag to track sorting state *)
END_VAR

(* Helper function to heapify a subtree rooted at index 'root' *)
METHOD Heapify : VOID
VAR_INPUT
    root : INT; (* Root index of subtree *)
    size : INT; (* Current heap size *)
END_VAR
VAR
    left : INT; (* Left child index *)
    right : INT; (* Right child index *)
    largest : INT; (* Index of largest value *)
    temp : INT; (* Temporary variable for swapping *)
    continue : BOOL; (* Flag to continue heapifying *)
END_VAR
(* Initialize *)
largest := root;
continue := TRUE;

(* Iterative heapify *)
WHILE continue DO
    (* Calculate child indices *)
    left := 2 * largest;
    right := 2 * largest + 1;
    
    (* Find largest among root, left, and right children *)
    IF left <= size AND InputArray[left] > InputArray[largest] THEN
        largest := left;
    END_IF;
    
    IF right <= size AND InputArray[right] > InputArray[largest] THEN
        largest := right;
    END_IF;
    
    (* Swap with largest child if needed and continue heapifying *)
    IF largest <> root THEN
        temp := InputArray[root];
        InputArray[root] := InputArray[largest];
        InputArray[largest] := temp;
        root := largest; (* Continue with the affected subtree *)
    ELSE
        continue := FALSE; (* Heap property satisfied *)
    END_IF;
END_WHILE;
END_METHOD

(* Reset outputs when not executing *)
IF NOT Execute THEN
    Done := FALSE;
    Fault := FALSE;
    Sorting := FALSE;
    FOR i := 1 TO ArraySize DO
        SortedArray[i] := InputArray[i]; (* Copy input to output *)
    END_FOR;
    RETURN;
END_IF;

(* Input validation *)
IF Execute AND NOT Sorting THEN
    IF ArraySize < 1 OR ArraySize > 100 THEN
        Fault := TRUE;
        Done := FALSE;
        Sorting := FALSE;
        RETURN;
    END_IF;
    
    (* Initialize sorting *)
    Sorting := TRUE;
    Done := FALSE;
    Fault := FALSE;
    HeapSize := ArraySize;
    
    (* Copy InputArray to SortedArray for in-place sorting *)
    FOR i := 1 TO ArraySize DO
        SortedArray[i] := InputArray[i];
    END_FOR;
END_IF;

(* Main heapsort logic *)
IF Sorting THEN
    (* Phase 1: Build max heap *)
    IF NOT Done THEN
        (* Start from last non-leaf node: floor(n/2) *)
        FOR i := HeapSize / 2 TO 1 BY -1 DO
            THIS.Heapify(i, HeapSize);
        END_FOR;
        
        (* Phase 2: Sorting *)
        FOR j := HeapSize TO 2 BY -1 DO
            (* Move root (max) to end *)
            Temp := SortedArray[1];
            SortedArray[1] := SortedArray[j];
            SortedArray[j] := Temp;
            
            (* Reduce heap size and heapify root *)
            HeapSize := HeapSize - 1;
            THIS.Heapify(1, HeapSize);
        END_FOR;
        
        (* Sorting complete *)
        Done := TRUE;
        Sorting := FALSE;
    END_IF;
END_IF;

(* Copy array to output if fault occurs *)
IF Fault THEN
    FOR i := 1 TO ArraySize DO
        SortedArray[i] := InputArray[i];
    END_FOR;
END_IF;

(* Performance Notes *)
(* - Iterative heapify avoids recursion, ensuring single-cycle or bounded execution per call. *)
(* - In-place sorting minimizes memory usage, critical for PLCs with limited resources. *)
(* - Ascending index loops comply with PLC standards, avoiding DOWNTO for compatibility. *)
(* - Bounded iterations (n/2 for heap construction, n for sorting) ensure predictable scan-cycle impact. *)
(* - Suitable for industrial applications like sorting sensor data or process setpoints. *)
END_FUNCTION_BLOCK
