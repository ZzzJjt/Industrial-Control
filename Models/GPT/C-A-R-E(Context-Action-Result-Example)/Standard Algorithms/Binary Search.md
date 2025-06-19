FUNCTION_BLOCK BinarySearchFB
VAR_INPUT
    Enable : BOOL;                           // Start signal
    TargetValue : INT;                       // Value to search for
    SearchArray : ARRAY[1..100] OF INT;      // Sorted input array
END_VAR
VAR_OUTPUT
    Index : INT;                             // Index of found value (1..100), or -1 if not found
    Found : BOOL;                            // TRUE if value found
END_VAR
VAR
    Low : INT;
    High : INT;
    Mid : INT;
END_VAR

IF Enable THEN
    Low := 1;
    High := 100;
    Found := FALSE;
    Index := -1;

    WHILE (Low <= High) AND NOT Found DO
        Mid := (Low + High) / 2;

        IF SearchArray[Mid] = TargetValue THEN
            Index := Mid;
            Found := TRUE;
        ELSIF SearchArray[Mid] > TargetValue THEN
            High := Mid - 1;
        ELSE
            Low := Mid + 1;
        END_IF;
    END_WHILE;

    IF NOT Found THEN
        Index := -1;
    END_IF;
ELSE
    Index := -1;
    Found := FALSE;
END_IF;
