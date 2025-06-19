FUNCTION_BLOCK FB_QuickSort
VAR_INPUT
    InputArray : ARRAY[1..100] OF INT; // Input array to sort (max 100 elements)
    Execute : BOOL;                    // Triggers sorting process
    N : INT;                           // Number of valid elements in InputArray
END_VAR

VAR_OUTPUT
    SortedArray : ARRAY[1..100] OF INT; // Optional: Sorted output
    Done : BOOL;                        // TRUE when sorting complete
    Busy : BOOL;                        // TRUE while sorting
END_VAR

VAR
    LowStack  : ARRAY[1..100] OF INT;  // Stack for low bounds
    HighStack : ARRAY[1..100] OF INT;  // Stack for high bounds
    StackPtr  : INT := 0;              // Stack pointer
    i, j      : INT;                   // Index variables
    Pivot     : INT;                   // Pivot value
    Temp      : INT;                   // Temporary variable for swapping
    Low, High : INT;                   // Current bounds
    PartIndex : INT;                   // Partition return value
    State     : INT := 0;              // Execution state
    Index     : INT;                   // Loop index
END_VAR

// === Main State Machine ===
CASE State OF

    0: // IDLE, wait for Execute
        IF Execute THEN
            // Copy input to working array
            FOR Index := 1 TO N DO
                SortedArray[Index] := InputArray[Index];
            END_FOR

            // Initialize stack with full array bounds
            StackPtr := 1;
            LowStack[StackPtr] := 1;
            HighStack[StackPtr] := N;

            Busy := TRUE;
            Done := FALSE;
            State := 1;
        END_IF

    1: // MAIN LOOP
        IF StackPtr > 0 THEN
            // Pop bounds from stack
            Low := LowStack[StackPtr];
            High := HighStack[StackPtr];
            StackPtr := StackPtr - 1;

            IF Low < High THEN
                // === Lomuto Partition ===
                Pivot := SortedArray[High];
                i := Low - 1;

                FOR j := Low TO High - 1 DO
                    IF SortedArray[j] <= Pivot THEN
                        i := i + 1;
                        // Swap SortedArray[i] <-> SortedArray[j]
                        Temp := SortedArray[i];
                        SortedArray[i] := SortedArray[j];
                        SortedArray[j] := Temp;
                    END_IF
                END_FOR

                // Final swap to place pivot correctly
                Temp := SortedArray[i + 1];
                SortedArray[i + 1] := SortedArray[High];
                SortedArray[High] := Temp;
                PartIndex := i + 1;

                // Push right side to stack
                IF PartIndex + 1 < High THEN
                    StackPtr := StackPtr + 1;
                    LowStack[StackPtr] := PartIndex + 1;
                    HighStack[StackPtr] := High;
                END_IF

                // Push left side to stack
                IF Low < PartIndex - 1 THEN
                    StackPtr := StackPtr + 1;
                    LowStack[StackPtr] := Low;
                    HighStack[StackPtr] := PartIndex - 1;
                END_IF
            END_IF
        ELSE
            // Done when stack is empty
            Busy := FALSE;
            Done := TRUE;
            State := 0; // Reset for next run
        END_IF

END_CASE
