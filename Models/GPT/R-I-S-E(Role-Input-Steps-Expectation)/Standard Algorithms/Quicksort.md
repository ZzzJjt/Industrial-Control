FUNCTION_BLOCK FB_QuickSort
VAR_INPUT
    Execute     : BOOL;
    InputArray  : ARRAY[1..100] OF INT; // Adjust size as needed
END_VAR

VAR_OUTPUT
    SortedArray : ARRAY[1..100] OF INT; // Sorted output (optional copy)
    Done        : BOOL;
END_VAR

VAR
    StackLow    : ARRAY[1..100] OF INT;
    StackHigh   : ARRAY[1..100] OF INT;
    StackPtr    : INT := 0;
    Low, High   : INT;
    Pivot, i, j : INT;
    Temp        : INT;
    Step        : INT := 0;
    Index       : INT;
    N           : INT := 100; // Array size constant
END_VAR

IF Execute AND Step = 0 THEN
    // Initialize
    FOR Index := 1 TO N DO
        SortedArray[Index] := InputArray[Index];
    END_FOR;

    StackPtr := 1;
    StackLow[StackPtr] := 1;
    StackHigh[StackPtr] := N;
    Done := FALSE;
    Step := 1;
END_IF

IF Step = 1 AND StackPtr > 0 THEN
    // Pop range from stack
    Low := StackLow[StackPtr];
    High := StackHigh[StackPtr];
    StackPtr := StackPtr - 1;

    // Partition
    Pivot := SortedArray[High];
    i := Low - 1;

    FOR j := Low TO High - 1 DO
        IF SortedArray[j] <= Pivot THEN
            i := i + 1;
            // Swap
            Temp := SortedArray[i];
            SortedArray[i] := SortedArray[j];
            SortedArray[j] := Temp;
        END_IF
    END_FOR

    // Swap pivot
    Temp := SortedArray[i + 1];
    SortedArray[i + 1] := SortedArray[High];
    SortedArray[High] := Temp;

    // Push subranges to stack
    IF (i > Low) THEN
        StackPtr := StackPtr + 1;
        StackLow[StackPtr] := Low;
        StackHigh[StackPtr] := i;
    END_IF

    IF (i + 2 < High) THEN
        StackPtr := StackPtr + 1;
        StackLow[StackPtr] := i + 2;
        StackHigh[StackPtr] := High;
    END_IF
END_IF

IF StackPtr = 0 AND Step = 1 THEN
    Done := TRUE;
    Step := 0;
END_IF

QuickSortFB : FB_QuickSort;

QuickSortFB.Execute := StartSort;
QuickSortFB.InputArray := MyInputArray; // Your input array

QuickSortFB(); // Call the FB

IF QuickSortFB.Done THEN
    SortedResult := QuickSortFB.SortedArray;
END_IF
