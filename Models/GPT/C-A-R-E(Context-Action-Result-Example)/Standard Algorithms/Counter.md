FUNCTION_BLOCK FB_DigitalCounter
VAR_INPUT
    Enable      : BOOL;     // Enables counting operation
    Reset       : BOOL;     // Resets to initial value
    CountUp     : BOOL;     // TRUE for up-counting, FALSE for down-counting
    InitValue   : INT;      // Initial value to load on reset
    StepSize    : INT;      // Increment or decrement step
    MaxValue    : INT;      // Maximum allowed value
    MinValue    : INT;      // Minimum allowed value
END_VAR

VAR_OUTPUT
    CurrentValue : INT;     // Current counter value
    AtMax        : BOOL;    // TRUE when value reaches MaxValue
    AtMin        : BOOL;    // TRUE when value reaches MinValue
END_VAR

VAR
    TempValue : INT;        // Temporary variable for calculation
END_VAR

// Main logic
IF Enable THEN
    IF Reset THEN
        CurrentValue := InitValue;
        AtMax := FALSE;
        AtMin := FALSE;

    ELSE
        // Determine direction
        IF CountUp THEN
            TempValue := CurrentValue + StepSize;
        ELSE
            TempValue := CurrentValue - StepSize;
        END_IF;

        // Check overflow/underflow and update value
        IF TempValue >= MaxValue THEN
            CurrentValue := MaxValue;
            AtMax := TRUE;
            AtMin := FALSE;
        ELSIF TempValue <= MinValue THEN
            CurrentValue := MinValue;
            AtMin := TRUE;
            AtMax := FALSE;
        ELSE
            CurrentValue := TempValue;
            AtMax := FALSE;
            AtMin := FALSE;
        END_IF;
    END_IF;
END_IF;
