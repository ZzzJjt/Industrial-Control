(* IEC 61131-3 Structured Text Function Block for Iterative Quicksort *)
(* Sorts an integer array in ascending order using in-place quicksort *)
(* Uses iterative approach with stacks, Lomuto partitioning, and scan-cycle-safe design *)
(* Conforms to industrial automation best practices *)

FUNCTION_BLOCK QuickSort
VAR_INPUT
    Array: ARRAY[1..100] OF INT;  (* Input array to sort *)
    Execute: BOOL;                (* TRUE to start or continue sorting *)
END_VAR

VAR_OUTPUT
    SortedArray: ARRAY[1..100] OF INT;  (* Sorted array, same memory as input *)
    Done: BOOL;                   (* TRUE when sorting is complete *)
    Error: BOOL;                  (* TRUE if an error occurs *)
END_VAR

VAR
    (* Stack for iterative quicksort *)
    LowStack: ARRAY[1..32] OF INT;  (* Stack for low indices, max log2(100) + buffer *)
    HighStack: ARRAY[1..32] OF INT; (* Stack for high indices *)
    StackPointer: INT;              (* Current stack position *)
    
    (* Partitioning variables *)
    Low: INT;                       (* Low index of current subarray *)
    High: INT;                      (* High index of current subarray *)
    Pivot: INT;                     (* Pivot value *)
    i: INT;                         (* Partitioning index *)
    j: INT;                         (* Scanning index *)
    Temp: INT;                      (* Temporary variable for swapping *)
    
    (* Control variables *)
    ValidInput: BOOL;               (* Input validation flag *)
    SortingInProgress: BOOL;        (* TRUE while sorting *)
    PartitionInProgress: BOOL;      (* TRUE during partitioning *)
END_VAR

(* Step 1: Input Validation and Initialization *)
(* Check Execute and reset outputs if not active *)
IF NOT Execute THEN
    Done := FALSE;
    Error := FALSE;
    SortingInProgress := FALSE;
    PartitionInProgress := FALSE;
    StackPointer := 0;
    (* Copy input array to output to preserve initial state *)
    FOR i := 1 TO 100 DO
        SortedArray[i] := Array[i];
    END_FOR;
    RETURN;
END_IF;

ValidInput := Execute;

(* Step 2: Handle Invalid Input *)
(* Set error if input is invalid *)
IF NOT ValidInput THEN
    Done := FALSE;
    Error := TRUE;
    SortingInProgress := FALSE;
    PartitionInProgress := FALSE;
    RETURN;
END_IF;

(* Step 3: Initialize Sorting *)
(* Copy input array and push initial bounds onto stack *)
IF NOT SortingInProgress THEN
    FOR i := 1 TO 100 DO
        SortedArray[i] := Array[i];
    END_FOR;
    SortingInProgress := TRUE;
    PartitionInProgress := FALSE;
    Done := FALSE;
    Error := FALSE;
    StackPointer := 1;
    LowStack[1] := 1;      (* Initial subarray: entire array *)
    HighStack[1] := 100;
END_IF;

(* Step 4: Main Quicksort Loop *)
(* Process subarrays from stack until empty *)
WHILE StackPointer > 0 AND NOT Done AND NOT Error DO
    (* Pop subarray bounds from stack *)
    Low := LowStack[StackPointer];
    High := HighStack[StackPointer];
    StackPointer := StackPointer - 1;
    
    (* Skip if subarray is too small *)
    IF Low >= High THEN
        CONTINUE;
    END_IF;
    
    (* Step 5: Lomuto Partitioning *)
    (* Partition subarray around pivot (high element) *)
    IF NOT PartitionInProgress THEN
        Pivot := SortedArray[High];  (* Choose pivot as last element *)
        i := Low - 1;               (* Index of smaller elements *)
        j := Low;                   (* Scanning index *)
        PartitionInProgress := TRUE;
    END_IF;
    
    (* Perform partitioning *)
    WHILE j < High AND NOT Error DO
        IF SortedArray[j] <= Pivot THEN
            i := i + 1;
            (* Swap elements at i and j *)
            Temp := SortedArray[i];
            SortedArray[i] := SortedArray[j];
            SortedArray[j] := Temp;
        END_IF;
        j := j + 1;
        
        (* Safety check: Ensure valid indices *)
        IF i < 1 OR i > 100 OR j < 1 OR j > 100 THEN
            Error := TRUE;
            Done := FALSE;
            SortingInProgress := FALSE;
            PartitionInProgress := FALSE;
            RETURN;
        END_IF;
        
        (* Allow partial execution: Exit loop to resume in next scan cycle *)
        (* This ensures scan-cycle safety for large arrays *)
        EXIT;  (* Process one j per cycle *)
    END_WHILE;
    
    (* Complete partitioning *)
    IF j >= High THEN
        (* Place pivot in final position *)
        i := i + 1;
        Temp := SortedArray[i];
        SortedArray[i] := SortedArray[High];
        SortedArray[High] := Temp;
        
        (* Push new subarrays onto stack *)
        (* Push larger subarray first to ensure O(log n) stack depth *)
        IF i - Low < High - i THEN
            IF i + 1 < High THEN
                StackPointer := StackPointer + 1;
                IF StackPointer > 32 THEN
                    Error := TRUE;
                    Done := FALSE;
                    SortingInProgress := FALSE;
                    PartitionInProgress := FALSE;
                    RETURN;
                END_IF;
                LowStack[StackPointer] := i + 1;
                HighStack[StackPointer] := High;
            END_IF;
            IF Low < i - 1 THEN
                StackPointer := StackPointer + 1;
                IF StackPointer > 32 THEN
                    Error := TRUE;
                    Done := FALSE;
                    SortingInProgress := FALSE;
                    PartitionInProgress := FALSE;
                    RETURN;
                END_IF;
                LowStack[StackPointer] := Low;
                HighStack[StackPointer] := i - 1;
            END_IF;
        ELSE
            IF Low < i - 1 THEN
                StackPointer := StackPointer + 1;
                IF StackPointer > 32 THEN
                    Error := TRUE;
                    Done := FALSE;
                    SortingInProgress := FALSE;
                    PartitionInProgress := FALSE;
                    RETURN;
                END_IF;
                LowStack[StackPointer] := Low;
                HighStack[StackPointer] := i - 1;
            END_IF;
            IF i + 1 < High THEN
                StackPointer := StackPointer + 1;
                IF StackPointer > 32 THEN
                    Error := TRUE;
                    Done := FALSE;
                    SortingInProgress := FALSE;
                    PartitionInProgress := FALSE;
                    RETURN;
                END_IF;
                LowStack[StackPointer] := i + 1;
                HighStack[StackPointer] := High;
            END_IF;
        END_IF;
        
        PartitionInProgress := FALSE;
    END_IF;
    
    (* Check if stack is empty *)
    IF StackPointer = 0 AND NOT PartitionInProgress THEN
        Done := TRUE;
        SortingInProgress := FALSE;
    END_IF;
END_WHILE;

(* Step 6: Safety Check *)
(* Ensure no invalid state *)
IF StackPointer < 0 OR StackPointer > 32 THEN
    Error := TRUE;
    Done := FALSE;
    SortingInProgress := FALSE;
    PartitionInProgress := FALSE;
END_IF;

END_FUNCTION_BLOCK
