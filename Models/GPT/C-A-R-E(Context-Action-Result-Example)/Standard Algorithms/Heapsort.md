FUNCTION_BLOCK FB_HeapSort_Int
VAR_INPUT
    Execute     : BOOL;
    InputArray  : ARRAY[1..100] OF INT;  // Input array of up to 100 elements
    Length      : INT;                   // Actual length of the array to sort
END_VAR

VAR_OUTPUT
    SortedArray : ARRAY[1..100] OF INT;  // Sorted output array
    Done        : BOOL;
END_VAR

VAR
    I, J        : INT;
    HeapSize    : INT;
    Temp        : INT;
    Child       : INT;
    Root        : INT;
    Swapped     : BOOL;
END_VAR

// Utility: Swap two elements
METHOD PRIVATE Swap : VOID
VAR_INPUT
    Idx1, Idx2 : INT;
END_VAR
VAR
    T : INT;
END_VAR
T := SortedArray[Idx1];
SortedArray[Idx1] := SortedArray[Idx2];
SortedArray[Idx2] := T;

// Main Sorting Logic
IF Execute AND NOT Done THEN
    // Step 1: Copy input
    FOR I := 1 TO Length DO
        SortedArray[I] := InputArray[I];
    END_FOR;

    // Step 2: Build max heap
    FOR I := (Length / 2) TO 1 BY -1 DO
        Root := I;
        Swapped := TRUE;
        WHILE Swapped DO
            Swapped := FALSE;
            Child := 2 * Root;
            IF Child <= Length THEN
                IF (Child < Length) AND (SortedArray[Child] < SortedArray[Child + 1]) THEN
                    Child := Child + 1;
                END_IF;
                IF SortedArray[Root] < SortedArray[Child] THEN
                    Swap(Idx1 := Root, Idx2 := Child);
                    Root := Child;
                    Swapped := TRUE;
                END_IF;
            END_IF;
        END_WHILE;
    END_FOR;

    // Step 3: Extract from heap
    HeapSize := Length;
    FOR I := Length TO 2 BY -1 DO
        Swap(Idx1 := 1, Idx2 := I);
        HeapSize := HeapSize - 1;

        Root := 1;
        Swapped := TRUE;
        WHILE Swapped DO
            Swapped := FALSE;
            Child := 2 * Root;
            IF Child <= HeapSize THEN
                IF (Child < HeapSize) AND (SortedArray[Child] < SortedArray[Child + 1]) THEN
                    Child := Child + 1;
                END_IF;
                IF SortedArray[Root] < SortedArray[Child] THEN
                    Swap(Idx1 := Root, Idx2 := Child);
                    Root := Child;
                    Swapped := TRUE;
                END_IF;
            END_IF;
        END_WHILE;
    END_FOR;

    Done := TRUE;
END_IF;
