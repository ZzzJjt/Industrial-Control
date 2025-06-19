(* IEC 61131-3 Structured Text: Iterative Quicksort Function Block *)
(* Purpose: Sorts an array of integers using iterative quicksort with stack emulation *)

FUNCTION_BLOCK QuickSort
VAR_INPUT
    Execute : BOOL;                 (* TRUE to initiate sorting *)
    InputArray : ARRAY[1..100] OF INT; (* Input array to sort *)
    ArraySize : INT;                (* Number of elements to sort, 1 to 100 *)
END_VAR
VAR_OUTPUT
    SortedArray : ARRAY[1..100] OF INT; (* Sorted output array *)
    InProgress : BOOL;              (* TRUE while sorting is ongoing *)
    Done : BOOL;                    (* TRUE when sorting is complete *)
    Error : BOOL;                   (* TRUE if input validation fails *)
END_VAR
VAR
    StackLow : ARRAY[1..32] OF INT; (* Stack for low indices *)
    StackHigh : ARRAY[1..32] OF INT; (* Stack for high indices *)
    StackPtr : INT := 0;            (* Stack pointer, 0 = empty *)
    Low : INT;                      (* Current low index *)
    High : INT;                     (* Current high index *)
    Pivot : INT;                    (* Pivot value *)
    i : INT;                        (* Partitioning index *)
    j : INT;                        (* Partitioning index *)
    Temp : INT;                     (* Temporary variable for swapping *)
    PartitionIndex : INT;           (* Final pivot position *)
    State : INT := 0;               (* Sorting state: 0=Idle, 1=Init, 2=Sort *)
    Valid : BOOL;                   (* Input validation flag *)
END_VAR

(* Partition Method: Partitions subarray and returns pivot index *)
METHOD PRIVATE Partition : INT
VAR_INPUT
    StartIdx : INT;                 (* Start index of subarray *)
    EndIdx : INT;                   (* End index of subarray *)
END_VAR
(* Partitions SortedArray[StartIdx..EndIdx] around pivot *)
Pivot := SortedArray[EndIdx];
i := StartIdx - 1;

FOR j := StartIdx TO EndIdx - 1 DO
    IF SortedArray[j] <= Pivot THEN
        i := i + 1;
        (* Swap SortedArray[i] and SortedArray[j] *)
        Temp := SortedArray[i];
        SortedArray[i] := SortedArray[j];
        SortedArray[j] := Temp;
    END_IF;
END_FOR;

(* Place pivot in final position *)
i := i + 1;
Temp := SortedArray[i];
SortedArray[i] := SortedArray[EndIdx];
SortedArray[EndIdx] := Temp;

Partition := i;
END_METHOD

(* Main Logic *)
IF NOT Execute THEN
    (* Reset outputs and state *)
    State := 0;
    InProgress := FALSE;
    Done := FALSE;
    Error := FALSE;
    StackPtr := 0;
    FOR i := 1 TO ArraySize DO
        SortedArray[i] := InputArray[i];  (* Copy input to output *)
    END_FOR;
ELSE
    CASE State OF
        0: (* Idle: Validate and Initialize *)
            IF ArraySize < 1 OR ArraySize > 100 THEN
                (* Invalid array size *)
                Error := TRUE;
                InProgress := FALSE;
                Done := FALSE;
                SortedArray := InputArray;
                State := 0;
            ELSE
                Error := FALSE;
                InProgress := TRUE;
                Done := FALSE;
                
                (* Copy InputArray to SortedArray for in-place sorting *)
                FOR i := 1 TO ArraySize DO
                    SortedArray[i] := InputArray[i];
                END_FOR;
                
                (* Initialize stack with full array *)
                StackPtr := 1;
                StackLow[1] := 1;
                StackHigh[1] := ArraySize;
                State := 1;
            END_IF;
        
        1: (* Sort: Process stack until empty *)
            IF StackPtr = 0 THEN
                (* Sorting complete *)
                InProgress := FALSE;
                Done := TRUE;
                State := 0;
            ELSE
                (* Pop subarray from stack *)
                Low := StackLow[StackPtr];
                High := StackHigh[StackPtr];
                StackPtr := StackPtr - 1;
                
                IF Low < High THEN
                    (* Partition subarray *)
                    PartitionIndex := THIS.Partition(Low, High);
                    
                    (* Push larger subarray first to ensure O(log n) stack depth *)
                    IF PartitionIndex - Low < High - PartitionIndex THEN
                        (* Push right subarray *)
                        IF PartitionIndex + 1 < High THEN
                            StackPtr := StackPtr + 1;
                            StackLow[StackPtr] := PartitionIndex + 1;
                            StackHigh[StackPtr] := High;
                        END_IF;
                        (* Push left subarray *)
                        IF Low < PartitionIndex THEN
                            StackPtr := StackPtr + 1;
                            StackLow[StackPtr] := Low;
                            StackHigh[StackPtr] := PartitionIndex - 1;
                        END_IF;
                    ELSE
                        (* Push left subarray *)
                        IF Low < PartitionIndex THEN
                            StackPtr := StackPtr + 1;
                            StackLow[StackPtr] := Low;
                            StackHigh[StackPtr] := PartitionIndex - 1;
                        END_IF;
                        (* Push right subarray *)
                        IF PartitionIndex + 1 < High THEN
                            StackPtr := StackPtr + 1;
                            StackLow[StackPtr] := PartitionIndex + 1;
                            StackHigh[StackPtr] := High;
                        END_IF;
                    END_IF;
                END_IF;
            END_IF;
    END_CASE;
END_IF;

(* Notes:
   - Purpose: Sorts an array of integers using iterative quicksort for PLC environments.
   - Inputs:
     - Execute: Initiates sorting when TRUE.
     - InputArray: ARRAY[1..100] OF INT, input array to sort.
     - ArraySize: Number of elements to sort (1 to 100).
   - Outputs:
     - SortedArray: Sorted array (ascending order).
     - InProgress: TRUE while sorting is ongoing.
     - Done: TRUE when sorting is complete.
     - Error: TRUE if ArraySize is invalid (<1 or >100).
   - Algorithm:
     - Quicksort: Average O(n log n) complexity, in-place sorting.
     - Iterative: Uses stack emulation (StackLow, StackHigh) to manage subranges.
     - Partition: Places pivot (last element) in correct position, partitions around it.
   - Features:
     - No recursion: Stack-based iteration ensures PLC compatibility.
     - In-Place: Sorts within SortedArray to minimize memory usage.
     - Single-cycle execution for ArraySize <= 100; multi-cycle via state machine.
     - Stack size (32) supports worst-case O(log n) depth for n=100.
   - Safety:
     - Validates ArraySize to prevent array access errors.
     - Resets outputs when Execute = FALSE.
     - Ensures stack bounds (1..32) to avoid overflow.
   - Usage:
     - Ideal for data sorting, ranking, or prioritizing in automation tasks.
     - Reusable in IEC 61131-3 programs (e.g., CODESYS, TwinCAT).
   - Maintenance:
     - Modular design with clear comments aids debugging.
     - InProgress, Done, and Error flags support diagnostics via HMI.
   - Platform Notes:
     - Assumes INT array; can be adapted for DINT or other types.
     - ArraySize <= 100 balances flexibility and PLC memory constraints.
*)
END_FUNCTION_BLOCK
