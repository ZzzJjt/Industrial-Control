FUNCTION_BLOCK BinarySearch
VAR_INPUT
    SearchArray : ARRAY[1..100] OF INT; // Sorted integer array to search
    TargetValue : INT;                  // Value to search for
    Enable : BOOL;                      // Enable signal to start the search
END_VAR

VAR_OUTPUT
    FoundIndex : INT := -1;            // Index of the found value (-1 if not found)
    Found : BOOL := FALSE;             // Flag indicating if the target was found
END_VAR

VAR
    Low : INT := 1;                    // Lower bound of the search window
    High : INT := 100;                 // Upper bound of the search window
    Mid : INT;                         // Middle point of the search window
    Searching : BOOL := FALSE;         // Internal flag to indicate ongoing search
END_VAR

METHOD Execute;
VAR
    ContinueSearch : BOOL := TRUE;     // Local flag to control the search loop
END_VAR
BEGIN
    // Reset output values
    FoundIndex := -1;
    Found := FALSE;

    // Validate input and check if search should be initiated
    IF Enable AND NOT Searching THEN
        Searching := TRUE;
        Low := 1;
        High := 100;
        ContinueSearch := TRUE;
    ELSIF NOT Enable THEN
        Searching := FALSE;
        RETURN;
    END_IF;

    // Perform binary search
    WHILE ContinueSearch DO
        Mid := (Low + High) DIV 2;

        IF Low > High THEN
            ContinueSearch := FALSE;
        ELSIF SearchArray[Mid] = TargetValue THEN
            FoundIndex := Mid;
            Found := TRUE;
            ContinueSearch := FALSE;
        ELSIF SearchArray[Mid] < TargetValue THEN
            Low := Mid + 1;
        ELSE
            High := Mid - 1;
        END_IF;
    END_WHILE;

    // Clear searching flag after completion
    Searching := FALSE;
END_METHOD



