FUNCTION_BLOCK FB_DigitalCounter
VAR_INPUT
    Enable    : BOOL;  // Enables counting on rising edge
    CountUp   : BOOL;  // TRUE = Count Up, FALSE = Count Down
    StepSize  : INT;   // Step size for each count
    InitValue : INT;   // Initial value after reset
    Reset     : BOOL;  // Active-high reset signal
    MaxValue  : INT;   // Upper limit
    MinValue  : INT;   // Lower limit
END_VAR

VAR_OUTPUT
    CurrentValue : INT := 0;   // Current counter value
    AtMax        : BOOL := FALSE; // TRUE if at or above MaxValue
    AtMin        : BOOL := FALSE; // TRUE if at or below MinValue
END_VAR

VAR
    LastEnable : BOOL := FALSE;  // For rising-edge detection
END_VAR

// Reset logic
IF Reset THEN
    CurrentValue := InitValue;
    AtMax := FALSE;
    AtMin := FALSE;
ELSE
    // Rising edge detection for Enable
    IF Enable AND NOT LastEnable THEN
        IF CountUp THEN
            // Increase counter
            IF (CurrentValue + StepSize) > MaxValue THEN
                CurrentValue := MaxValue;
                AtMax := TRUE;
                AtMin := FALSE;
            ELSE
                CurrentValue := CurrentValue + StepSize;
                AtMax := (CurrentValue = MaxValue);
                AtMin := FALSE;
            END_IF
        ELSE
            // Decrease counter
            IF (CurrentValue - StepSize) < MinValue THEN
                CurrentValue := MinValue;
                AtMin := TRUE;
                AtMax := FALSE;
            ELSE
                CurrentValue := CurrentValue - StepSize;
                AtMin := (CurrentValue = MinValue);
                AtMax := FALSE;
            END_IF
        END_IF
    END_IF
END_IF

// Save current enable state for edge detection
LastEnable := Enable;
