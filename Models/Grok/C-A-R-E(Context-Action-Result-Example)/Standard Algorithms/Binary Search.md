FUNCTION_BLOCK BinarySearchFB
VAR_INPUT
    Enable: BOOL;                     // Enable search operation
    SearchArray: ARRAY[1..100] OF INT; // Sorted array of integers
    TargetValue: INT;                 // Value to search for
END_VAR

VAR_OUTPUT
    Index: INT := -1;                 // Index of found value (-1 if not found)
    Found: BOOL := FALSE;             // True if target value is found
    Error: BOOL := FALSE;             // True if input validation fails
END_VAR

VAR
    Low: INT := 1;                    // Lower bound of search range
    High: INT := 100;                 // Upper bound of search range
    Mid: INT;                         // Middle index of search range
    ValidInput: BOOL;                 // Input validation flag
    i: INT;                           // Loop variable for array validation
END_VAR

// Reset outputs on disable
IF NOT Enable THEN
    Index := -1;
    Found := FALSE;
    Error := FALSE;
    Low := 1;
    High := 100;
    RETURN;
END_IF;

// Input validation
ValidInput := TRUE;

// Check array bounds
IF High > 100 OR Low < 1 THEN
    ValidInput := FALSE;
    Error := TRUE;
END_IF;

// Verify array is sorted (ascending order)
FOR i := 1 TO 99 DO
    IF SearchArray[i] > SearchArray[i + 1] THEN
        ValidInput := FALSE;
        Error := TRUE;
        EXIT;
    END_IF;
END_FOR;

// Perform binary search if inputs are valid
IF ValidInput THEN
    Found := FALSE;
    Index := -1;

    WHILE (Low <= High) AND NOT Found AND NOT Error DO
        // Calculate middle index (avoid overflow)
        Mid := Low + (High - Low) / 2;

        // Check for out-of-bounds access
        IF Mid < 1 OR Mid > 100 THEN
            Error := TRUE;
            EXIT;
        END_IF;

        IF SearchArray[Mid] = TargetValue THEN
            // Target found
            Index := Mid;
            Found := TRUE;
        ELSIF SearchArray[Mid] > TargetValue THEN
            // Search lower half
            High := Mid - 1;
        ELSE
            // Search upper half
            Low := Mid + 1;
        END_IF;
    END_WHILE;

    // If not found, ensure Index is -1
    IF NOT Found THEN
        Index := -1;
    END_IF;
ELSE
    // Set error outputs on invalid input
    Index := -1;
    Found := FALSE;
    Error := TRUE;
END_IF;

// Reset internal variables for next cycle
IF NOT Enable OR Found OR Error THEN
    Low := 1;
    High := 100;
END_IF;
END_FUNCTION_BLOCK
