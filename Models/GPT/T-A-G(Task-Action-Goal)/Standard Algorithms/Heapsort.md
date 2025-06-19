FUNCTION_BLOCK FB_HeapSort
VAR_INPUT
    Execute     : BOOL;                        // Trigger sorting when TRUE
    InputArray  : ARRAY[1..100] OF INT;        // Input array to sort
    Length      : INT := 100;                  // Number of elements to sort
END_VAR

VAR_OUTPUT
    SortedArray : ARRAY[1..100] OF INT;        // Output: sorted in ascending order
    Done        : BOOL;                        // TRUE when sorting is complete
END_VAR

VAR
    i, j, n     : INT;
    Temp        : INT;
    State       : INT := 0;
    HeapifyIdx  : INT;
    MaxIdx      : INT;
    Left        : INT;
    Right       : INT;
    Sorting     : BOOL := FALSE;
    HeapBuilt   : BOOL := FALSE;
END_VAR

// Copy input only once at start
IF Execute AND NOT Sorting THEN
    FOR i := 1 TO Length DO
        SortedArray[i] := InputArray[i];
    END_FOR;
    Sorting := TRUE;
    n := Length;
    HeapifyIdx := n / 2;  // Start from middle node for heapify
    Done := FALSE;
END_IF

IF Sorting THEN
    // Step 1: Build Max Heap
    IF NOT HeapBuilt THEN
        IF HeapifyIdx >= 1 THEN
            i := HeapifyIdx;
            MaxIdx := i;
            Left := 2 * i;
            Right := 2 * i + 1;

            // Find largest among node and children
            IF Left <= n AND SortedArray[Left] > SortedArray[MaxIdx] THEN
                MaxIdx := Left;
            END_IF;

            IF Right <= n AND SortedArray[Right] > SortedArray[MaxIdx] THEN
                MaxIdx := Right;
            END_IF;

            // Swap and reset heapify index if needed
            IF MaxIdx <> i THEN
                Temp := SortedArray[i];
                SortedArray[i] := SortedArray[MaxIdx];
                SortedArray[MaxIdx] := Temp;

                HeapifyIdx := MaxIdx; // Reheapify from swapped child
            ELSE
                HeapifyIdx := HeapifyIdx - 1;
            END_IF;
        ELSE
            HeapBuilt := TRUE;
            i := n;
        END_IF

    // Step 2: Sorting phase
    ELSE
        IF i > 1 THEN
            // Swap root (max) with last node
            Temp := SortedArray[1];
            SortedArray[1] := SortedArray[i];
            SortedArray[i] := Temp;

            // Reduce heap size
            i := i - 1;
            HeapifyIdx := 1;
            MaxIdx := HeapifyIdx;
            HeapBuilt := FALSE; // Rebuild heap with smaller size
            n := i;
        ELSE
            Done := TRUE;
            Sorting := FALSE;
        END_IF
    END_IF
END_IF
