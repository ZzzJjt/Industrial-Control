FUNCTION_BLOCK FB_QuickSort
VAR_INPUT
    Execute     : BOOL;                      // Start sorting trigger
    InputArray  : ARRAY[1..100] OF INT;      // Input array (modifiable)
    Size        : INT;                       // Number of elements to sort
END_VAR

VAR_OUTPUT
    Done        : BOOL;                      // Sorting complete
    SortedArray : ARRAY[1..100] OF INT;      // Sorted result
END_VAR

VAR
    LowStack    : ARRAY[1..100] OF INT;      // Simulated call stack (low indices)
    HighStack   : ARRAY[1..100] OF INT;      // Simulated call stack (high indices)
    StackPtr    : INT := 0;                  // Stack pointer
    Low, High   : INT;                       // Current subarray bounds
    Pivot, Temp : INT;
    i, j        : INT;
    SwapNeeded  : BOOL;
    Sorting     : BOOL := FALSE;             // Internal flag for sort in progress
    k           : INT;
END_VAR

// ===== INITIALIZATION =====
IF Execute AND NOT Sorting THEN
    FOR k := 1 TO Size DO
        SortedArray[k] := InputArray[k];     // Copy input to internal array
    END_FOR;

    StackPtr := 1;
    LowStack[StackPtr] := 1;
    HighStack[StackPtr] := Size;

    Sorting := TRUE;
    Done := FALSE;
END_IF

// ===== SORTING PROCESS =====
IF Sorting THEN
    IF StackPtr > 0 THEN
        Low := LowStack[StackPtr];
        High := HighStack[StackPtr];
        StackPtr := StackPtr - 1;

        // Partitioning using Lomuto scheme
        Pivot := SortedArray[High];
        i := Low - 1;

        FOR j := Low TO High - 1 DO
            IF SortedArray[j] <= Pivot THEN
                i := i + 1;
                // Swap SortedArray[i] and SortedArray[j]
                Temp := SortedArray[i];
                SortedArray[i] := SortedArray[j];
                SortedArray[j] := Temp;
            END_IF;
        END_FOR;

        // Swap pivot into correct position
        Temp := SortedArray[i + 1];
        SortedArray[i + 1] := SortedArray[High];
        SortedArray[High] := Temp;

        // Push subarrays onto stack
        IF i > Low THEN
            StackPtr := StackPtr + 1;
            LowStack[StackPtr] := Low;
            HighStack[StackPtr] := i;
        END_IF;

        IF i + 2 < High THEN
            StackPtr := StackPtr + 1;
            LowStack[StackPtr] := i + 2;
            HighStack[StackPtr] := High;
        END_IF;
    ELSE
        Done := TRUE;
        Sorting := FALSE;
    END_IF
END_IF
