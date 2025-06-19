FUNCTION_BLOCK HeapSortFB
VAR_INPUT
    Execute: BOOL;                     // Trigger sorting operation
    InputArray: ARRAY[1..100] OF INT;  // Input array to sort
    ArraySize: INT := 100;             // Size of array (1 to 100)
END_VAR

VAR_OUTPUT
    SortedArray: ARRAY[1..100] OF INT; // Sorted array (in-place)
    Done: BOOL;                        // TRUE when sorting is complete
    Error: BOOL;                       // TRUE if invalid input
END_VAR

VAR
    HeapSize: INT;                     // Current heap size
    i: INT;                            // Loop variable
    Temp: INT;                         // Temporary variable for swapping
    ValidInput: BOOL;                  // Input validation flag
    SortingInProgress: BOOL;           // Tracks sorting state
END_VAR

// Internal procedure for heapify (iterative)
METHOD Heapify : BOOL
VAR_INPUT
    Index: INT;                        // Root index to heapify
    Size: INT;                         // Current heap size
END_VAR
VAR
    Largest: INT;                      // Index of largest value
    Left: INT;                         // Left child index
    Right: INT;                        // Right child index
    Current: INT;                      // Current index for iteration
    SwapTemp: INT;                     // Temporary variable for swapping
    Continue: BOOL;                    // Continue heapify loop
END_VAR

Current := Index;
Continue := TRUE;

WHILE Continue DO
    Largest := Current;
    Left := 2 * Current;
    Right := 2 * Current + 1;

    // Compare with left child
    IF Left <= Size AND InputArray[Left] > InputArray[Largest] THEN
        Largest := Left;
    END_IF;

    // Compare with right child
    IF Right <= Size AND InputArray[Right] > InputArray[Largest] THEN
        Largest := Right;
    END_IF;

    // Swap with largest child and continue if needed
    IF Largest <> Current THEN
        SwapTemp := InputArray[Current];
        InputArray[Current] := InputArray[Largest];
        InputArray[Largest] := SwapTemp;
        Current := Largest; // Move down to largest child
    ELSE
        Continue := FALSE; // Heap property restored
    END_IF;
END_WHILE;

Heapify := TRUE;
END_METHOD

// Main function block logic
// Reset outputs when not executing
IF NOT Execute THEN
    Done := FALSE;
    Error := FALSE;
    SortingInProgress := FALSE;
    FOR i := 1 TO ArraySize DO
        SortedArray[i] := InputArray[i]; // Initialize output
    END_FOR;
    RETURN;
END_IF;

// Input validation
ValidInput := TRUE;
IF ArraySize < 1 OR ArraySize > 100 THEN
    Error := TRUE;
    Done := FALSE;
    SortingInProgress := FALSE;
    RETURN;
END_IF;

// Copy input to output for in-place sorting
IF NOT SortingInProgress THEN
    FOR i := 1 TO ArraySize DO
        SortedArray[i] := InputArray[i];
    END_FOR;
    SortingInProgress := TRUE;
    HeapSize := ArraySize;
END_IF;

// Build max heap
IF SortingInProgress AND NOT Done THEN
    // Start from last non-leaf node (ArraySize / 2)
    FOR i := HeapSize / 2 TO 1 BY -1 DO
        // Adjust index for ascending loop
        IF NOT Heapify(HeapSize - i + 1, HeapSize) THEN
            Error := TRUE;
            Done := FALSE;
            SortingInProgress := FALSE;
            RETURN;
        END_IF;
    END_FOR;

    // Extract elements from heap
    FOR i := HeapSize TO 2 BY -1 DO
        // Swap root (max) with last element
        Temp := SortedArray[1];
        SortedArray[1] := SortedArray[i];
        SortedArray[i] := Temp;
        HeapSize := HeapSize - 1;

        // Heapify root
        IF NOT Heapify(1, HeapSize) THEN
            Error := TRUE;
            Done := FALSE;
            SortingInProgress := FALSE;
            RETURN;
        END_IF;
    END_FOR;

    // Sorting complete
    Done := TRUE;
    SortingInProgress := FALSE;
END_IF;
END_FUNCTION_BLOCK
