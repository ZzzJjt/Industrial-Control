FUNCTION_BLOCK FB_BinarySearch
VAR_INPUT
    SearchArray : ARRAY[1..100] OF INT;  // Sorted array
    TargetValue : INT;                   // Value to find
    Enable : BOOL;                       // Rising edge trigger
END_VAR

VAR_OUTPUT
    Found : BOOL := FALSE;               // TRUE if value found
    Index : INT := -1;                   // Index of found value (1..100), or -1 if not found
    Done : BOOL := FALSE;                // TRUE when search completes
END_VAR

VAR
    Low : INT := 1;
    High : INT := 100;
    Mid : INT;
    State : INT := 0;                    // 0: Idle, 1: Searching
    RisingEdge : R_TRIG;                 // Detect Enable rising edge
END_VAR

// Detect rising edge of Enable
RisingEdge(CLK := Enable);

CASE State OF

    0: // Idle
        Found := FALSE;
        Done := FALSE;
        Index := -1;
        IF RisingEdge.Q THEN
            Low := 1;
            High := 100;
            State := 1; // Begin search
        END_IF;

    1: // Searching
        IF Low > High THEN
            Done := TRUE;
            Found := FALSE;
            Index := -1;
            State := 0;
        ELSE
            Mid := (Low + High) / 2;
            IF SearchArray[Mid] = TargetValue THEN
                Found := TRUE;
                Index := Mid;
                Done := TRUE;
                State := 0;
            ELSIF SearchArray[Mid] < TargetValue THEN
                Low := Mid + 1;
            ELSE
                High := Mid - 1;
            END_IF;
        END_IF;

END_CASE;
