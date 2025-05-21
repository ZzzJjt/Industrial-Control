(* IEC 61131-3 Structured Text Function Block for Heapsort *)
(* Sorts an integer array in ascending order using in-place heapsort *)
(* Uses iterative loops, avoids recursion and DOWNTO for scan-cycle safety *)
(* Conforms to industrial automation best practices *)

FUNCTION_BLOCK HeapSort
VAR_INPUT
    Array: ARRAY[1..100] OF INT;  (* Input array to sort *)
    Enable: BOOL;                 (* TRUE to start sorting *)
END_VAR

VAR_OUTPUT
    SortedArray: ARRAY[1..100] OF INT;  (* Sorted array, same memory as input *)
    Done: BOOL;                   (* TRUE when sorting is complete *)
    Error: BOOL;                  (* TRUE if an error occurs *)
END_VAR

VAR
    HeapSize: INT;                (* Current size of the heap *)
    i: INT;                       (* Loop index *)
    Parent: INT;                  (* Parent node index *)
    LeftChild: INT;               (* Left child node index *)
    RightChild: INT;              (* Right child node index *)
    Largest: INT;                 (* Index of largest value in heapify *)
    Temp: INT;                    (* Temporary variable for swapping *)
    ValidInput: BOOL;             (* Input validation flag *)
    SortingInProgress: BOOL;      (* Flag to track sorting state *)
END_VAR

(* Step 1: Input Validation and Initialization *)
(* Check Enable and initialize outputs *)
IF NOT Enable THEN
    Done := FALSE;
    Error := FALSE;
    SortingInProgress := FALSE;
    (* Copy input array to output to ensure initial state *)
    FOR i := 1 TO 100 DO
        SortedArray[i] := Array[i];
    END_FOR;
    RETURN;
END_IF;

ValidInput := Enable;

(* Step 2: Handle Invalid Input *)
(* Set error if input is invalid *)
IF NOT ValidInput THEN
    Done := FALSE;
    Error := TRUE;
    SortingInProgress := FALSE;
    RETURN;
END_IF;

(* Step 3: Initialize Sorting *)
(* Copy input array to output and start heap construction *)
IF NOT SortingInProgress THEN
    FOR i := 1 TO 100 DO
        SortedArray[i] := Array[i];
    END_FOR;
    HeapSize := 100;  (* Initial heap size *)
    SortingInProgress := TRUE;
    Done := FALSE;
    Error := FALSE;
END_IF;

(* Step 4: Heap Construction Phase *)
(* Build max-heap by heapifying from last non-leaf node to root *)
(* Last non-leaf node is at index HeapSize / 2 *)
FOR i := HeapSize / 2 TO 1 BY -1 DO
    (* Iterative heapify for subtree rooted at index i *)
    Parent := i;
    WHILE TRUE DO
        LeftChild := 2 * Parent;      (* Left child index *)
        RightChild := LeftChild + 1;  (* Right child index *)
        Largest := Parent;            (* Assume parent is largest *)
        
        (* Compare with left child, if it exists *)
        IF LeftChild <= HeapSize AND SortedArray[LeftChild] > SortedArray[Largest] THEN
            Largest := LeftChild;
        END_IF;
        
        (* Compare with right child, if it exists *)
        IF RightChild <= HeapSize AND SortedArray[RightChild] > SortedArray[Largest] THEN
            Largest := RightChild;
        END_IF;
        
        (* If parent is largest, heap property is satisfied *)
        IF Largest = Parent THEN
            EXIT;  (* Break inner WHILE loop *)
        END_IF;
        
        (* Swap parent with largest child *)
        Temp := SortedArray[Parent];
        SortedArray[Parent] := SortedArray[Largest];
        SortedArray[Largest] := Temp;
        
        (* Continue heapifying down the affected subtree *)
        Parent := Largest;
        
        (* Safety check: Prevent infinite loop *)
        IF Parent > HeapSize THEN
            Error := TRUE;
            Done := FALSE;
            SortingInProgress := FALSE;
            RETURN;
        END_IF;
    END_WHILE;
END_FOR;

(* Step 5: Sorting Phase *)
(* Extract largest element, swap to end, and heapify reduced heap *)
FOR i := 1 TO 99 DO  (* Process until one element remains *)
    (* Swap root (largest) with last element of heap *)
    Temp := SortedArray[1];
    SortedArray[1] := SortedArray[HeapSize];
    SortedArray[HeapSize] := Temp;
    
    (* Reduce heap size *)
    HeapSize := HeapSize - 1;
    
    (* Heapify the root of the reduced heap *)
    Parent := 1;
    WHILE TRUE DO
        LeftChild := 2 * Parent;
        RightChild := LeftChild + 1;
        Largest := Parent;
        
        (* Compare with left child *)
        IF LeftChild <= HeapSize AND SortedArray[LeftChild] > SortedArray[Largest] THEN
            Largest := LeftChild;
        END_IF;
        
        (* Compare with right child *)
        IF RightChild <= HeapSize AND SortedArray[RightChild] > SortedArray[Largest] THEN
            Largest := RightChild;
        END_IF;
        
        (* If parent is largest, heap property is satisfied *)
        IF Largest = Parent THEN
            EXIT;
        END_IF;
        
        (* Swap parent with largest child *)
        Temp := SortedArray[Parent];
        SortedArray[Parent] := SortedArray[Largest];
        SortedArray[Largest] := Temp;
        
        (* Continue heapifying down *)
        Parent := Largest;
        
        (* Safety check *)
        IF Parent > HeapSize THEN
            Error := TRUE;
            Done := FALSE;
            SortingInProgress := FALSE;
            RETURN;
        END_IF;
    END_WHILE;
END_FOR;

(* Step 6: Finalize Sorting *)
(* When heap size is 1, array is sorted *)
IF HeapSize = 1 THEN
    Done := TRUE;
    SortingInProgress := FALSE;
END_IF;

(* Step 7: Safety Check *)
(* Ensure no invalid array access occurred *)
IF HeapSize < 1 OR HeapSize > 100 THEN
    Error := TRUE;
    Done := FALSE;
    SortingInProgress := FALSE;
END_IF;

END_FUNCTION_BLOCK
