FUNCTION_BLOCK HeapsortFunctionBlock
VAR_INPUT
    InputArray : ARRAY[1..100] OF INT; // Input array to be sorted
    Enable : BOOL;                     // Enable signal to start sorting
END_VAR

VAR_OUTPUT
    SortedArray : ARRAY[1..100] OF INT; // Output sorted array
    Done : BOOL;                        // Flag indicating sorting is complete
END_VAR

VAR
    n : INT;                            // Size of the array
    i, j : INT;                         // Loop indices
    Largest : INT;                      // Index of the largest element
    Temp : INT;                         // Temporary variable for swapping
    Heapifying : BOOL := FALSE;         // Flag to indicate heapification process
END_VAR

METHOD Execute;
BEGIN
    // Initialize outputs
    SortedArray := InputArray;
    Done := FALSE;

    // Only proceed if enabled
    IF Enable THEN
        n := SIZEOF(InputArray);

        // Build max heap (rearrange array)
        FOR i := n DIV 2 DOWNTO 1 BY -1 DO
            CALL Heapiy(i);
        END_FOR;

        // One by one extract elements from heap
        FOR i := n DOWNTO 2 BY -1 DO
            // Move current root to end
            Temp := SortedArray[1];
            SortedArray[1] := SortedArray[i];
            SortedArray[i] := Temp;

            // Call max heapify on the reduced heap
            n := n - 1;
            CALL Heapiy(1);
        END_FOR;

        // Set done flag
        Done := TRUE;
    END_IF;
END_METHOD

METHOD Heapiy(IN index : INT);
VAR
    LeftChild, RightChild : INT;
BEGIN
    Largest := index; // Initialize largest as root
    LeftChild := 2 * index; // left = 2*index
    RightChild := 2 * index + 1; // right = 2*index + 1

    // If left child is larger than root
    IF LeftChild <= n AND SortedArray[LeftChild] > SortedArray[Largest] THEN
        Largest := LeftChild;
    END_IF;

    // If right child is larger than largest so far
    IF RightChild <= n AND SortedArray[RightChild] > SortedArray[Largest] THEN
        Largest := RightChild;
    END_IF;

    // If largest is not root
    IF Largest <> index THEN
        // Swap
        Temp := SortedArray[index];
        SortedArray[index] := SortedArray[Largest];
        SortedArray[Largest] := Temp;

        // Recursively heapify the affected sub-tree
        CALL Heapiy(Largest);
    END_IF;
END_METHOD

// Replace Heapiy method with iterative version
METHOD HeapiyIterative(IN index : INT);
VAR
    Largest : INT;
    LeftChild, RightChild : INT;
    CurrentIndex : INT;
BEGIN
    CurrentIndex := index;

    REPEAT
        Largest := CurrentIndex;
        LeftChild := 2 * CurrentIndex;
        RightChild := 2 * CurrentIndex + 1;

        // If left child is larger than root
        IF LeftChild <= n AND SortedArray[LeftChild] > SortedArray[Largest] THEN
            Largest := LeftChild;
        END_IF;

        // If right child is larger than largest so far
        IF RightChild <= n AND SortedArray[RightChild] > SortedArray[Largest] THEN
            Largest := RightChild;
        END_IF;

        // If largest is not root
        IF Largest <> CurrentIndex THEN
            // Swap
            Temp := SortedArray[CurrentIndex];
            SortedArray[CurrentIndex] := SortedArray[Largest];
            SortedArray[Largest] := Temp;

            // Move to the next node to heapify
            CurrentIndex := Largest;
        ELSE
            EXIT;
        END_IF;
    UNTIL FALSE;
END_METHOD

// Modify Execute method to use iterative Heapiy
METHOD Execute;
BEGIN
    // Initialize outputs
    SortedArray := InputArray;
    Done := FALSE;

    // Only proceed if enabled
    IF Enable THEN
        n := SIZEOF(InputArray);

        // Build max heap (rearrange array)
        FOR i := n DIV 2 TO 1 BY -1 DO
            CALL HeapiyIterative(i);
        END_FOR;

        // One by one extract elements from heap
        FOR i := n TO 2 BY -1 DO
            // Move current root to end
            Temp := SortedArray[1];
            SortedArray[1] := SortedArray[i];
            SortedArray[i] := Temp;

            // Call max heapify on the reduced heap
            n := n - 1;
            CALL HeapiyIterative(1);
        END_FOR;

        // Set done flag
        Done := TRUE;
    END_IF;
END_METHOD



