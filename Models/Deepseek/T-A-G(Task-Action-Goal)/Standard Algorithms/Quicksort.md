FUNCTION_BLOCK QuicksortFunctionBlock
VAR_INPUT
    InputArray : ARRAY[1..100] OF INT; // Input array to be sorted
    Execute : BOOL;                    // Signal to start sorting
END_VAR

VAR_OUTPUT
    SortedArray : ARRAY[1..100] OF INT; // Output sorted array
    Done : BOOL;                       // Flag indicating sorting is complete
END_VAR

VAR
    LowStack : ARRAY[1..50] OF INT;     // Stack for low indices
    HighStack : ARRAY[1..50] OF INT;    // Stack for high indices
    StackPointer : INT := 0;            // Pointer for the stack
    n : INT;                            // Size of the array
    i, j : INT;                         // Loop indices
    Pivot : INT;                        // Pivot element
    Temp : INT;                         // Temporary variable for swapping
    Low, High : INT;                    // Current subarray bounds
END_VAR

METHOD Execute;
BEGIN
    // Initialize outputs
    SortedArray := InputArray;
    Done := FALSE;

    // Only proceed if enabled
    IF Execute THEN
        n := SIZEOF(InputArray);

        // Initialize stack with the entire array
        StackPointer := 1;
        LowStack[StackPointer] := 1;
        HighStack[StackPointer] := n;

        // Process the stack until it's empty
        WHILE StackPointer > 0 DO
            // Pop the top subarray bounds from the stack
            Low := LowStack[StackPointer];
            High := HighStack[StackPointer];
            StackPointer := StackPointer - 1;

            // Partition the subarray
            CALL Partition(Low, High);

            // Push the left subarray bounds onto the stack if it exists
            IF Low < (i - 1) THEN
                StackPointer := StackPointer + 1;
                LowStack[StackPointer] := Low;
                HighStack[StackPointer] := i - 1;
            END_IF;

            // Push the right subarray bounds onto the stack if it exists
            IF (i + 1) < High THEN
                StackPointer := StackPointer + 1;
                LowStack[StackPointer] := i + 1;
                HighStack[StackPointer] := High;
            END_IF;
        END_WHILE;

        // Set done flag
        Done := TRUE;
    END_IF;
END_METHOD

METHOD Partition(IN Low : INT; IN High : INT);
BEGIN
    // Choose the rightmost element as pivot
    Pivot := SortedArray[High];

    // Index of smaller element
    i := Low - 1;

    // Traverse through all elements in the subarray
    FOR j := Low TO High - 1 DO
        // If current element is smaller than or equal to pivot
        IF SortedArray[j] <= Pivot THEN
            // Increment index of smaller element
            i := i + 1;

            // Swap SortedArray[i] and SortedArray[j]
            Temp := SortedArray[i];
            SortedArray[i] := SortedArray[j];
            SortedArray[j] := Temp;
        END_IF;
    END_FOR;

    // Swap SortedArray[i + 1] and SortedArray[High] (or pivot)
    Temp := SortedArray[i + 1];
    SortedArray[i + 1] := SortedArray[High];
    SortedArray[High] := Temp;

    // Return the partitioning index
    i := i + 1;
END_METHOD



