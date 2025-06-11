FUNCTION_BLOCK BinarySearchFB
VAR_INPUT
    SearchArray : ARRAY[1..100] OF INT; // Sorted array of integers
    TargetValue : INT;                  // Value to search for
    Enable : BOOL;                      // Trigger to start the search
END_VAR

VAR_OUTPUT
    Found : BOOL;                       // Flag indicating if the target was found
    Index : INT;                        // Index of the target value or -1 if not found
END_VAR

VAR
    Low : INT := 1;
    High : INT := 100;
    Mid : INT;
    ValidInput : BOOL;
END_VAR

METHOD Execute : VOID
BEGIN
    // Reset output values
    Found := FALSE;
    Index := -1;

    // Input validation
    IF NOT Enable THEN
        RETURN;
    END_IF;

    ValidInput := TRUE;

    FOR i := 1 TO 99 DO
        IF SearchArray[i] > SearchArray[i + 1] THEN
            ValidInput := FALSE;
            EXIT;
        END_IF;
    END_FOR;

    IF NOT ValidInput THEN
        RETURN;
    END_IF;

    // Binary search logic
    WHILE Low <= High AND NOT Found DO
        Mid := (Low + High) DIV 2;

        IF SearchArray[Mid] = TargetValue THEN
            Found := TRUE;
            Index := Mid;
        ELSIF SearchArray[Mid] < TargetValue THEN
            Low := Mid + 1;
        ELSE
            High := Mid - 1;
        END_IF;
    END_WHILE;
END_METHOD

END_FUNCTION_BLOCK
