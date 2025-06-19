FUNCTION_BLOCK FB_Heapsort
VAR_INPUT
    InputArray : ARRAY[1..100] OF INT; // Input array to be sorted
    Execute : BOOL;                    // Triggers the sorting operation
    N : INT;                           // Number of elements to sort (<= 100)
END_VAR

VAR_OUTPUT
    SortedArray : ARRAY[1..100] OF INT; // Sorted output array (same size as input)
    Done : BOOL;                        // TRUE when sorting is complete
END_VAR

VAR
    HeapSize : INT;
    i, j, k, Largest, Temp : INT;
    State : INT := 0;
    Index : INT;
    BuildPhase : BOOL := FALSE;
    SortPhase : BOOL := FALSE;
    SwapRequired : BOOL := FALSE;
    HeapifyIndex : INT;
END_VAR

// === Helper Procedure to Maintain Max Heap Property Iteratively ===
METHOD PRIVATE Heapify : VOID
VAR_INPUT
    StartIndex : INT;
    Size : INT;
END_VAR
VAR
    Root : INT;
    Child : INT;
    SwapIndex : INT;
    Continue : BOOL;
BEGIN
    Root := StartIndex;
    Continue := TRUE;
    WHILE Continue DO
        Child := 2 * Root;
        IF Child > Size THEN
            Continue := FALSE; // No children, done
        ELSE
            // Check if right child exists and is greater
            IF (Child + 1 <= Size) AND (SortedArray[Child + 1] > SortedArray[Child]) THEN
                Child := Child + 1;
            END_IF

            // Compare root with largest child
            IF SortedArray[Root] < SortedArray[Child] THEN
                // Swap root and child
                Temp := SortedArray[Root];
                SortedArray[Root] := SortedArray[Child];
                SortedArray[Child] := Temp;
                Root := Child; // Move down the tree
            ELSE
                Continue := FALSE; // Heap property satisfied
            END_IF
        END_IF
    END_WHILE
END_METHOD

// === Main Sorting Logic ===
IF Execute AND (State = 0) THEN
    // Copy input to working array
    FOR Index := 1 TO N DO
        SortedArray[Index] := InputArray[Index];
    END_FOR
    HeapSize := N;
    i := HeapSize / 2; // Start from middle
    State := 1;
END_IF

// === Build Max Heap Phase ===
IF State = 1 THEN
    IF i >= 1 THEN
        Heapify(StartIndex := i, Size := HeapSize);
        i := i - 1;
    ELSE
        State := 2;
        i := HeapSize;
    END_IF
END_IF

// === Sorting Phase ===
IF State = 2 THEN
    IF i > 1 THEN
        // Swap root (max) with last element
        Temp := SortedArray[1];
        SortedArray[1] := SortedArray[i];
        SortedArray[i] := Temp;

        HeapSize := HeapSize - 1;
        Heapify(StartIndex := 1, Size := HeapSize);
        i := i - 1;
    ELSE
        Done := TRUE;
        State := 0; // Reset for next run
    END_IF
END_IF
