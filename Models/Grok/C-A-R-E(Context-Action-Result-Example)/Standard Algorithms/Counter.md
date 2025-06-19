FUNCTION_BLOCK DigitalCounterFB
VAR_INPUT
    Enable: BOOL;           // Enable counter operation
    Reset: BOOL;            // Reset counter to initial value
    CountUp: BOOL;          // TRUE for up, FALSE for down
    InitValue: DINT;        // Initial counter value
    StepSize: DINT := 1;    // Step size for counting
    MaxValue: DINT := 9999; // Upper limit
    MinValue: DINT := -9999;// Lower limit
END_VAR

VAR_OUTPUT
    CurrentValue: DINT;     // Current counter value
    AtMax: BOOL;            // TRUE if at max limit
    AtMin: BOOL;            // TRUE if at min limit
    Error: BOOL;            // TRUE if configuration error
END_VAR

VAR
    ValidConfig: BOOL;      // Configuration validation flag
END_VAR

// Reset outputs on disable
IF NOT Enable THEN
    CurrentValue := InitValue;
    AtMax := FALSE;
    AtMin := FALSE;
    Error := FALSE;
    RETURN;
END_IF;

// Validate configuration
ValidConfig := TRUE;
IF StepSize <= 0 OR MaxValue <= MinValue OR InitValue > MaxValue OR InitValue < MinValue THEN
    ValidConfig := FALSE;
    Error := TRUE;
    CurrentValue := InitValue;
    AtMax := FALSE;
    AtMin := FALSE;
    RETURN;
END_IF;

// Counter logic
IF ValidConfig THEN
    // Handle reset
    IF Reset THEN
        CurrentValue := InitValue;
        AtMax := FALSE;
        AtMin := FALSE;
    ELSE
        // Perform counting
        IF CountUp THEN
            // Up counting
            IF CurrentValue <= MaxValue - StepSize THEN
                CurrentValue := CurrentValue + StepSize;
                AtMax := FALSE;
            ELSE
                CurrentValue := MaxValue;
                AtMax := TRUE;
            END_IF;
        ELSE
            // Down counting
            IF CurrentValue >= MinValue + StepSize THEN
                CurrentValue := CurrentValue - StepSize;
                AtMin := FALSE;
            ELSE
                CurrentValue := MinValue;
                AtMin := TRUE;
            END_IF;
        END_IF;
    END_IF;
END_IF;
END_FUNCTION_BLOCK
