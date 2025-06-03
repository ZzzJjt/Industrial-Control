FUNCTION_BLOCK FB_FlexibleCounter
VAR_INPUT
    Enable     : BOOL;       // Activates counting
    CountUp    : BOOL;       // TRUE = count up, FALSE = count down
    StepSize   : INT := 1;   // Step size per count
    InitValue  : INT := 0;   // Initial value
    Reset      : BOOL;       // Resets counter to InitValue
    MaxValue   : INT := 100; // Upper limit
    MinValue   : INT := 0;   // Lower limit
END_VAR

VAR_OUTPUT
    CurrentValue : INT;      // Current counter value
    AtMax        : BOOL;     // TRUE if CurrentValue >= MaxValue
    AtMin        : BOOL;     // TRUE if CurrentValue <= MinValue
END_VAR

VAR
    LastEnable   : BOOL := FALSE; // To detect rising edge
END_VAR

// Reset logic
IF Reset THEN
    CurrentValue := InitValue;
END_IF

// Rising edge detection for Enable
IF Enable AND NOT LastEnable THEN
    IF CountUp THEN
        CurrentValue := CurrentValue + StepSize;
        IF CurrentValue > MaxValue THEN
            CurrentValue := MaxValue;
        END_IF
    ELSE
        CurrentValue := CurrentValue - StepSize;
        IF CurrentValue < MinValue THEN
            CurrentValue := MinValue;
        END_IF
    END_IF
END_IF

// Update status flags
AtMax := (CurrentValue >= MaxValue);
AtMin := (CurrentValue <= MinValue);

// Save previous Enable state
LastEnable := Enable;
