FUNCTION_BLOCK ConfigurableDigitalCounter
VAR_INPUT
    Enable : BOOL;          // Activates the counter
    CountUp : BOOL;         // Selects counting direction (TRUE = Up, FALSE = Down)
    StepSize : INT := 1;     // Amount to increment or decrement
    InitValue : INT := 0;    // Starting value of the counter
    Reset : BOOL;           // Resets the counter to the initial value
    MaxValue : INT := 100;   // Upper bound for overflow handling
    MinValue : INT := 0;     // Lower bound for underflow handling
END_VAR

VAR_OUTPUT
    CurrentValue : INT;      // The current counter state
    AtMax : BOOL;            // Flag indicating if the counter is at the maximum value
    AtMin : BOOL;            // Flag indicating if the counter is at the minimum value
END_VAR

VAR
    LastReset : BOOL := FALSE; // Internal flag to detect reset edge
    Initialized : BOOL := FALSE; // Internal flag to ensure initialization
END_VAR

METHOD Execute;
BEGIN
    // Initialize the counter on first call or when Reset is triggered
    IF NOT Initialized OR Reset AND NOT LastReset THEN
        CurrentValue := InitValue;
        Initialized := TRUE;
        AtMax := CurrentValue >= MaxValue;
        AtMin := CurrentValue <= MinValue;
    END_IF;

    // Update the last reset state
    LastReset := Reset;

    // Only proceed if the counter is enabled
    IF Enable THEN
        // Determine the new value based on CountUp direction
        IF CountUp THEN
            CurrentValue := CurrentValue + StepSize;
        ELSE
            CurrentValue := CurrentValue - StepSize;
        END_IF;

        // Clamp the value within the specified bounds
        IF CurrentValue > MaxValue THEN
            CurrentValue := MaxValue;
            AtMax := TRUE;
            AtMin := FALSE;
        ELSIF CurrentValue < MinValue THEN
            CurrentValue := MinValue;
            AtMax := FALSE;
            AtMin := TRUE;
        ELSE
            AtMax := FALSE;
            AtMin := FALSE;
        END_IF;
    END_IF;
END_METHOD



