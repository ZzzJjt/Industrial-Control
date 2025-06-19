FUNCTION_BLOCK FB_IterativeQuickSort
VAR_INPUT
    Execute     : BOOL;
    InputArray  : ARRAY[1..100] OF INT;
    Length      : INT; // Number of elements in the array
END_VAR

VAR_OUTPUT
    SortedArray : ARRAY[1..100] OF INT;
    Done        : BOOL;
END_VAR

VAR
    StackLow    : ARRAY[1..100] OF INT;
    StackHigh   : ARRAY[1..100] OF INT;
    Top         : INT := 0;
    Low         : INT;
    High        : INT;
    Pivot       : INT;
    i, j        : INT;
    Temp        : INT;
    Index       : INT;
    Sorting     : BOOL := FALSE;
END_VAR

// Copy input array to working array at first execution
IF Execute AND NOT Sorting THEN
    FOR Index := 1 TO Length DO
        SortedArray[Index] := InputArray[Index];
    END_FOR

    // Initialize stack with full array range
    Top := 1;
    StackLow[Top] := 1;
    StackHigh[Top] := Length;
    Sorting := TRUE;
    Done := FALSE;
END_IF

// Main iterative quicksort loop
IF Sorting THEN
    IF Top > 0 THEN
        // Pop the stack
        Low := StackLow[Top];
        High := StackHigh[Top];
        Top := Top - 1;

        // Partitioning logic
        Pivot := SortedArray[High];
        i := Low - 1;

        FOR j := Low TO High - 1 DO
            IF SortedArray[j] <= Pivot THEN
                i := i + 1;

                // Swap SortedArray[i] and SortedArray[j]
                Temp := SortedArray[i];
                SortedArray[i] := SortedArray[j];
                SortedArray[j] := Temp;
            END_IF
        END_FOR

        // Swap SortedArray[i+1] and SortedArray[High] (pivot)
        Temp := SortedArray[i + 1];
        SortedArray[i + 1] := SortedArray[High];
        SortedArray[High] := Temp;

        // Push subarrays to stack
        IF (i > Low) THEN
            Top := Top + 1;
            StackLow[Top] := Low;
            StackHigh[Top] := i;
        END_IF

        IF (i + 2 < High) THEN
            Top := Top + 1;
            StackLow[Top] := i + 2;
            StackHigh[Top] := High;
        END_IF
    ELSE
        Sorting := FALSE;
        Done := TRUE;
    END_IF
END_IF
