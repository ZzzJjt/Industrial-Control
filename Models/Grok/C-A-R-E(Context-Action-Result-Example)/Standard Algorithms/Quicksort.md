FUNCTION_BLOCK QuickSortFB
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
    LowStack: ARRAY[1..100] OF INT;    // Stack for low bounds
    HighStack: ARRAY[1..100] OF INT;   // Stack for high bounds
    StackPointer: INT := 0;            // Stack pointer
    Low: INT;                          // Current low index
    High: INT;                         // Current high index
    i: INT;                            // Loop variable
    Temp: INT;                         // Temporary variable for swapping
    ValidInput: BOOL;                  // Input validation flag
    SortingInProgress: BOOL;           // Tracks sorting state
END_VAR

// Internal method for partitioning (Lomuto scheme)
METHOD Partition : INT
VAR_INPUT
    Low: INT;                          // Low index of subarray
    High: INT;                         // High index of subarray
END_VAR
VAR
    Pivot: INT;                        // Pivot value
    i: INT;
