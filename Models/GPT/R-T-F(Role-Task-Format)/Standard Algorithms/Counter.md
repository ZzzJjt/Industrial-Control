FUNCTION_BLOCK FB_DigitalCounter
VAR_INPUT
    Enable      : BOOL;     // Enable counting when TRUE
    Up          : BOOL;     // TRUE = Count Up, FALSE = Count Down
    Reset       : BOOL;     // Resets count to InitialValue
    InitialValue: INT;      // Starting value of the counter
    StepSize    : INT := 1; // Count increment/decrement per scan
    MaxValue    : INT := 10000; // Maximum allowable value
    MinValue    : INT := 0;     // Minimum allowable value
END_VAR

VAR_OUTPUT
    Count       : INT;      // Current counter value
    AtMax       : BOOL;     // TRUE if counter reached MaxValue
    AtMin       : BOOL;     // TRUE if counter reached MinValue
END_VAR

VAR
    Count_Internal : INT;   // Internal counter variable
    FirstScan      : BOOL := TRUE; // Flag for initial setup
END_VAR

// === Initialization ===
IF FirstScan THEN
    Count_Internal := InitialValue;
    FirstScan := FALSE;
END_IF

// === Reset Logic ===
IF Reset THEN
    Count_Internal := InitialValue;
END_IF

// === Counting Logic ===
IF Enable AND NOT Reset THEN
    IF Up THEN
        IF (Count_Internal + StepSize) <= MaxValue THEN
            Count_Internal := Count_Internal + StepSize;
        ELSE
            Count_Internal := MaxValue; // Clamp to MaxValue
        END_IF;
    ELSE
        IF (Count_Internal - StepSize) >= MinValue THEN
            Count_Internal := Count_Internal - StepSize;
        ELSE
            Count_Internal := MinValue; // Clamp to MinValue
        END_IF;
    END_IF;
END_IF

// === Output Assignment ===
Count := Count_Internal;
AtMax := (Count_Internal >= MaxValue);
AtMin := (Count_Internal <= MinValue);
