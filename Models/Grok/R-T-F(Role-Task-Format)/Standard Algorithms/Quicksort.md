(* IEC 61131-3 Structured Text function block for quicksort algorithm *)
(* Sorts an array of integers in ascending order using iterative logic *)
(* Uses Lomuto partitioning and a stack to replace recursion *)
(* Optimized for PLC scan-cycle execution and in-place sorting *)

FUNCTION_BLOCK QuickSort
VAR_INPUT
    Execute : BOOL; (* TRUE to start sorting *)
    InputArray : ARRAY[1..100] OF INT; (* Array to sort *)
    ArraySize : INT := 100; (* Number of elements to sort, 1 to 100 *)
END_VAR

VAR_OUTPUT
    SortedArray : ARRAY[1..100] OF INT; (* Sorted array, same as InputArray *)
    Done : BOOL := FALSE; (* TRUE when sorting is complete *)
    Busy : BOOL := FALSE; (* TRUE while sorting is in progress *)
    Fault : BOOL := FALSE; (* TRUE if invalid inputs detected *)
END_VAR

VAR
    StackLow : ARRAY[0..31] OF INT; (* Stack for low indices *)
    StackHigh : ARRAY[0..31] OF INT; (* Stack for high indices *)
    StackTop : INT := -1; (* Top of stack index *)
    Low : INT; (* Current low index *)
    High : INT; (* Current high index *)
    PivotIndex : INT; (* Index of pivot after partitioning *)
    i : INT; (* Loop variable *)
    Temp : INT; (* Temporary variable for swapping *)
    SortingState : INT := 0; (* 0=Idle, 1=Init, 2=Partition, 3=StackPop *)
    Partition_i : INT; (* Partitioning loop variable *)
    Partition_j : INT; (* Partitioning swap index *)
    PivotValue : INT; (* Pivot value for partitioning *)
END_VAR

(* Reset outputs when not executing *)
IF NOT Execute THEN
    Done := FALSE;
    Busy := FALSE;
    Fault := FALSE;
    SortingState := 0;
    StackTop := -1;
    FOR i := 1 TO ArraySize DO
        SortedArray[i] := InputArray[i]; (* Copy input to output *)
    END_FOR;
    RETURN;
END_IF;

(* Main quicksort logic *)
CASE SortingState OF
    0: (* Idle - Validate inputs and initialize *)
        IF Execute AND NOT Busy THEN
            (* Validate array size *)
            IF ArraySize < 1 OR ArraySize > 100 THEN
                Fault := TRUE;
                Done := FALSE;
                Busy := FALSE;
                RETURN;
            END_IF;
            
            (* Initialize *)
            Busy := TRUE;
            Done := FALSE;
            Fault := FALSE;
            StackTop := -1;
            SortingState := 1;
            
            (* Copy InputArray to SortedArray for in-place sorting *)
            FOR i := 1 TO ArraySize DO
                SortedArray[i] := InputArray[i];
            END_FOR;
            
            (* Push initial bounds to stack *)
            StackTop := StackTop + 1;
            StackLow[StackTop] := 1;
            StackHigh[StackTop] := ArraySize;
        END_IF;
        
    1: (* Initialize partition *)
        IF StackTop >= 0 THEN
            (* Pop subarray bounds from stack *)
            Low := StackLow[StackTop];
            High := StackHigh[StackTop];
            StackTop := StackTop - 1;
            
            (* Skip small subarrays (size <= 1) *)
            IF Low < High THEN
                SortingState := 2; (* Proceed to partitioning *)
                Partition_i := Low - 1;
                Partition_j := Low;
                PivotValue := SortedArray[High];
            ELSE
                SortingState := 3; (* Check stack for more partitions *)
            END_IF;
        ELSE
            (* Stack empty, sorting complete *)
            Done := TRUE;
            Busy := FALSE;
            SortingState := 0;
        END_IF;
        
    2: (* Partition - Lomuto method *)
        (* Process one partition per cycle *)
        IF Partition_j < High THEN
            (* Compare element with pivot *)
            IF SortedArray[Partition_j] <= PivotValue THEN
                Partition_i := Partition_i + 1;
                Temp := SortedArray[Partition_i];
                SortedArray[Partition_i] := SortedArray[Partition_j];
                SortedArray[Partition_j] := Temp;
            END_IF;
            Partition_j := Partition_j + 1;
        ELSE
            (* Finalize partition *)
            Partition_i := Partition_i + 1;
            Temp := SortedArray[Partition_i];
            SortedArray[Partition_i] := SortedArray[High];
            SortedArray[High] := Temp;
            PivotIndex := Partition_i;
            
            (* Push subarrays to stack *)
            IF PivotIndex - 1 > Low AND StackTop < 31 THEN
                StackTop := StackTop + 1;
                StackLow[StackTop] := Low;
                StackHigh[StackTop] := PivotIndex - 1;
            END_IF;
            IF PivotIndex + 1 < High AND StackTop < 31 THEN
                StackTop := StackTop + 1;
                StackLow[StackTop] := PivotIndex + 1;
                StackHigh[StackTop] := High;
            END_IF;
            
            (* Check for stack overflow *)
            IF StackTop >= 31 THEN
                Fault := TRUE;
                Done := FALSE;
                Busy := FALSE;
                SortingState := 0;
                RETURN;
            END_IF;
            
            SortingState := 3; (* Check stack for next partition *)
        END_IF;
        
    3: (* Stack Pop - Process next partition *)
        IF StackTop >= 0 THEN
            SortingState := 1; (* Initialize next partition *)
        ELSE
            Done := TRUE;
            Busy := FALSE;
            SortingState := 0;
        END_IF;
END_CASE;

(* Execution Notes *)
(* - Iterative quicksort uses a stack to manage subarray bounds, avoiding recursion for PLC compatibility. *)
(* - Lomuto partitioning is efficient, with O(n) comparisons per partition. *)
(* - Processes one partition per scan cycle, ensuring scan-cycle safety for large arrays. *)
(* - Stack size limited to 32 entries (log2(100) ≈ 7), preventing overflow. *)
(* - In-place sorting minimizes memory usage, critical for PLCs with limited resources. *)
(* - Status flags (Busy, Done, Fault) enable integration with larger control systems. *)
END_FUNCTION_BLOCK
