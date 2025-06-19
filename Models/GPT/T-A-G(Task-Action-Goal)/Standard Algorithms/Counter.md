FUNCTION_BLOCK FB_DigitalCounter
VAR_INPUT
    Enable     : BOOL;   // Enable counting
    CountUp    : BOOL;   // TRUE = count up, FALSE = count down
    StepSize   : INT;    // Amount to increment/decrement
    InitValue  : INT;    // Initial/reset value
    Reset      : BOOL;   // Resets counter to InitValue
    MaxValue   : INT;    // Maximum allowable value
    MinValue   : INT;    // Minimum allowable value
END_VAR

VAR_OUTPUT
    CurrentValue : INT;   // Current counter state
    AtMax        : BOOL;  // TRUE if value hits MaxValue
    AtMin        : BOOL;  // TRUE if value hits MinValue
END_VAR

VAR
    PrevEnable : BOOL;    // Rising edge detection for Enable
END_VAR

// Rising edge detection for Enable
IF (Enable AND NOT PrevEnable) THEN

    // Reset takes priority
    IF Reset THEN
        CurrentValue := InitValue;

    ELSIF CountUp THEN
        // Count Up logic
        CurrentValue := CurrentValue + StepSize;
        IF CurrentValue > MaxValue THEN
            CurrentValue := MaxValue;
        END_IF;

    ELSE
        // Count Down logic
        CurrentValue := CurrentValue - StepSize;
        IF CurrentValue < MinValue THEN
            CurrentValue := MinValue;
        END_IF;

    END_IF;

END_IF;

// Set boundary flags
AtMax := (CurrentValue >= MaxValue);
AtMin := (CurrentValue <= MinValue);

// Update previous state
PrevEnable := Enable;
