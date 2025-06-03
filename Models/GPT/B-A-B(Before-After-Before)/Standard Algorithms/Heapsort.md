FUNCTION_BLOCK FB_HeapSort
VAR_INPUT
    Execute    : BOOL;
    InputArray : ARRAY[1..100] OF INT; // Modify size as needed
END_VAR

VAR_OUTPUT
    SortedArray : ARRAY[1..100] OF INT;
    Done        : BOOL;
END_VAR

VAR
    N          : INT := 100;
    HeapSize   : INT;
    I, J, Largest, Left, Right, Temp : INT;
    SortIndex  : INT;
    Step       : INT := 0;
    Sorting    : BOOL := FALSE;
END_VAR

// Utility: Swap elements
METHOD PRIVATE Swap : VOID
VAR_INPUT
    IndexA : INT;
    IndexB : INT;
END_VAR
VAR
    Tmp : INT;
END_VAR
Tmp := SortedArray[IndexA];
SortedArray[IndexA] := SortedArray[IndexB];
SortedArray[IndexB] := Tmp;
END_METHOD

// Utility: Heapify using iterative loop
METHOD PRIVATE Heapify : VOID
VAR_INPUT
    Root     : INT;
    Size     : INT;
END_VAR
VAR
    Current, MaxIndex : INT;
BEGIN
    Current := Root;
    WHILE TRUE DO
        Left := 2 * Current;
        Right := 2 * Current + 1;
        MaxIndex := Current;

        IF (Left <= Size) AND (SortedArray[Left] > SortedArray[MaxIndex]) THEN
            MaxIndex := Left;
        END_IF;
        IF (Right <= Size) AND (SortedArray[Right] > SortedArray[MaxIndex]) THEN
            MaxIndex := Right;
        END_IF;

        IF MaxIndex = Current THEN
            EXIT; // Heap property satisfied
        END_IF;

        Swap(Current, MaxIndex);
        Current := MaxIndex;
    END_WHILE;
END_METHOD

// Main Execution
IF Execute AND NOT Sorting THEN
    // Copy input to working array
    FOR I := 1 TO N DO
        SortedArray[I] := InputArray[I];
    END_FOR;
    HeapSize := N;
    Sorting := TRUE;
    Step := 0;
    Done := FALSE;
END_IF;

IF Sorting THEN
    CASE Step OF
        0: // Build heap
            I := N / 2;
            Step := 1;

        1: // Heapify from middle to top
            IF I >= 1 THEN
                Heapify(I, HeapSize);
                I := I - 1;
            ELSE
                Step := 2;
                SortIndex := N;
            END_IF;

        2: // Sort down
            IF SortIndex > 1 THEN
                Swap(1, SortIndex);
                HeapSize := HeapSize - 1;
                Heapify(1, HeapSize);
                SortIndex := SortIndex - 1;
            ELSE
                Step := 3;
            END_IF;

        3: // Done
            Sorting := FALSE;
            Done := TRUE;
    END_CASE;
END_IF;
