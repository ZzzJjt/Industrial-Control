FUNCTION_BLOCK DigitalCounterFB
VAR_INPUT
    Enable : BOOL;          // Enable counting
    CountUp : BOOL;         // Direction of count (TRUE for up, FALSE for down)
    StepSize : INT := 1;     // Step size for each count
    InitValue : INT := 0;    // Initial value of the counter
    Reset : BOOL;           // Reset the counter to InitValue
    MaxValue : INT := 100;   // Maximum value before overflow
    MinValue : INT := 0;     // Minimum value before underflow
END_VAR

VAR_OUTPUT
    CurrentValue : INT;      // Current value of the counter
    AtMax : BOOL;            // Flag indicating if the counter is at MaxValue
    AtMin : BOOL;            // Flag indicating if the counter is at MinValue
END_VAR

VAR
    LastReset : BOOL;
    LastEnable : BOOL;
END_VAR

METHOD Execute : VOID
BEGIN
    // Initialize flags
    AtMax := FALSE;
    AtMin := FALSE;

    // Handle reset condition
    IF Reset AND NOT LastReset THEN
        CurrentValue := InitValue;
    END_IF;
    LastReset := Reset;

    // Check enable and update counter
    IF Enable AND NOT LastEnable THEN
        IF CountUp THEN
            IF CurrentValue + StepSize <= MaxValue THEN
                CurrentValue := CurrentValue + StepSize;
            ELSE
                CurrentValue := MaxValue;
                AtMax := TRUE;
            END_IF;
        ELSE
            IF CurrentValue - StepSize >= MinValue THEN
                CurrentValue := CurrentValue - StepSize;
            ELSE
                CurrentValue := MinValue;
                AtMin := TRUE;
            END_IF;
        END_IF;
    END_IF;
    LastEnable := Enable;
END_METHOD

END_FUNCTION_BLOCK

PROGRAM MainProgram
VAR
    DCFB : DigitalCounterFB;
    CurrentValue : INT;
    AtMax : BOOL;
    AtMin : BOOL;
END_VAR

// Configure the counter
DCFB(Enable := TRUE,
     CountUp := TRUE,
     StepSize := 1,
     InitValue := 0,
     Reset := FALSE,
     MaxValue := 10,
     MinValue := 0);

// Call the function block
DCFB.Execute();

// Use outputs as needed
CurrentValue := DCFB.CurrentValue;
AtMax := DCFB.AtMax;
AtMin := DCFB.AtMin;

// Example usage: Print current value and status
// Note: In a real PLC environment, you would typically use outputs to control other parts of the system
IF AtMax THEN
    // Counter has reached maximum value
ELSIF AtMin THEN
    // Counter has reached minimum value
ELSE
    // Counter is within range
END_IF;
END_PROGRAM
