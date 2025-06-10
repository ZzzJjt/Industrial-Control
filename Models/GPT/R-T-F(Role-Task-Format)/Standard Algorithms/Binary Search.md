FUNCTION_BLOCK FB_BinarySearch_INT
VAR_INPUT
    Enable      : BOOL;                    // Triggers search execution
    Target      : INT;                     // Value to search for
    DataArray   : ARRAY [1..100] OF INT;   // Sorted array to search
END_VAR

VAR_OUTPUT
    Index       : INT := -1;               // Found index (1-based), -1 if not found
    Found       : BOOL := FALSE;           // True if value found
END_VAR

VAR
    Low         : INT := 1;                // Start of search range
    High        : INT := 100;              // End of search range
    Mid         : INT;
    Done        : BOOL := FALSE;           // Loop termination flag
END_VAR

// === Main Binary Search Logic ===
IF Enable AND NOT Done THEN
    Low := 1;
    High := 100;
    Index := -1;
    Found := FALSE;

    WHILE (Low <= High) AND NOT Found DO
        Mid := (Low + High) / 2;

        IF DataArray[Mid] = Target THEN
            Index := Mid;
            Found := TRUE;
        ELSIF DataArray[Mid] < Target THEN
            Low := Mid + 1;
        ELSE
            High := Mid - 1;
        END_IF;
    END_WHILE;

    Done := TRUE; // Prevent repeated execution until Enable toggled again
END_IF;

// === Reset for next enable cycle ===
IF NOT Enable THEN
    Done := FALSE;
END_IF;
