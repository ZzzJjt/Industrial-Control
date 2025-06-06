FUNCTION_BLOCK FB_QuickSort
VAR_INPUT
    Execute     : BOOL;                          // Start sorting
    InputArray  : ARRAY[1..100] OF INT;          // Array to sort (in-place)
    Length      : INT;                           // Number of valid elements in array
END_VAR

VAR_OUTPUT
    Done        : BOOL;                          // Sorting complete
    SortedArray : ARRAY[1..100] OF INT;          // Sorted output (optional copy)
END_VAR

VAR
    LowStack    : ARRAY[1..100] OF INT;
    HighStack   : ARRAY[1..100] OF INT;
    StackPointer: INT := 0;
    Low, High   : INT;
    Pivot, i, j : INT;
    Temp        : INT;
    Running     : BOOL := FALSE;
END_VAR

// Initialization
IF Execute AND NOT Running THEN
    SortedArray := InputArray;  // Copy input to output array
    StackPointer := 1;
    LowStack[1] := 1;
    HighStack[1] := Length;
    Running := TRUE;
    Done := FALSE;
END_IF

// Iterative Quicksort
IF Running THEN
    WHILE StackPointer > 0 DO
        Low := LowStack[StackPointer];
        High := HighStack[StackPointer];
        StackPointer := StackPointer - 1;

        IF Low < High THEN
            // Lomuto partitioning
            Pivot := SortedArray[High];
            i := Low - 1;

            FOR j := Low TO High - 1 DO
                IF SortedArray[j] <= Pivot THEN
                    i := i + 1;
                    Temp := SortedArray[i];
                    SortedArray[i] := SortedArray[j];
                    SortedArray[j] := Temp;
                END_IF;
            END_FOR;

            Temp := SortedArray[i + 1];
            SortedArray[i + 1] := SortedArray[High];
            SortedArray[High] := Temp;

            // Push subarrays to stack
            StackPointer := StackPointer + 1;
            LowStack[StackPointer] := Low;
            HighStack[StackPointer] := i;

            StackPointer := StackPointer + 1;
            LowStack[StackPointer] := i + 2;
            HighStack[StackPointer] := High;
        END_IF;
    END_WHILE;

    Running := FALSE;
    Done := TRUE;
END_IF;
