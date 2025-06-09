FUNCTION_BLOCK FB_BinarySearch
VAR_INPUT
    SearchArray : ARRAY[1..100] OF INT;   // Sorted input array
    TargetValue : INT;                    // Value to search for
    Enable      : BOOL;                   // Rising-edge trigger
END_VAR

VAR_OUTPUT
    Found : BOOL := FALSE;                // Output TRUE if value found
    Index : INT := -1;                    // Output index or -1 if not found
END_VAR

VAR
    Low        : INT := 1;
    High       : INT := 100;
    Mid        : INT;
    SearchDone : BOOL := FALSE;
    Trigger    : R_TRIG;
END_VAR

// Trigger detection
Trigger(CLK := Enable);

// Binary search logic on rising edge
IF Trigger.Q AND NOT SearchDone THEN
    Found := FALSE;
    Index := -1;
    Low := 1;
    High := 100;

    WHILE (Low <= High) DO
        Mid := (Low + High) / 2;
        IF SearchArray[Mid] = TargetValue THEN
            Found := TRUE;
            Index := Mid;
            EXIT;
        ELSIF SearchArray[Mid] < TargetValue THEN
            Low := Mid + 1;
        ELSE
            High := Mid - 1;
        END_IF;
    END_WHILE;

    SearchDone := TRUE;
END_IF;

// Reset when Enable goes low
IF NOT Enable THEN
    SearchDone := FALSE;
END_IF;
