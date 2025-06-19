FUNCTION_BLOCK FB_BinarySearch
VAR_INPUT
    SearchArray : ARRAY[1..100] OF INT;
    TargetValue : INT;
    Enable : BOOL;
END_VAR

VAR_OUTPUT
    Found : BOOL := FALSE;
    Index : INT := -1;
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
            Found := TRUE;
            Index := Mid;

        ELSIF SearchArray[Mid] < TargetValue THEN
            Low := Mid + 1;

        ELSE
            High := Mid - 1;
        END_IF;
    END_WHILE;
END_IF;
