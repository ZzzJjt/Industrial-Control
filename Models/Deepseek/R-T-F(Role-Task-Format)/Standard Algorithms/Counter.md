FUNCTION_BLOCK FB_Counter
VAR_INPUT
    Enable : BOOL; // Enable flag to start the counter
    CountUp : BOOL; // Direction flag: TRUE for up-counting, FALSE for down-counting
    Reset : BOOL; // Reset flag to reinitialize the counter
    StepSize : INT := 1; // Step size for each count increment/decrement
    MaxValue : INT := 32767; // Maximum value limit to prevent overflow
    MinValue : INT := -32768; // Minimum value limit to prevent underflow
END_VAR

VAR_OUTPUT
    CurrentCount : INT := 0; // Current count value
    Overflow : BOOL := FALSE; // Status flag indicating overflow condition
    Underflow : BOOL := FALSE; // Status flag indicating underflow condition
END_VAR

VAR
    InitialValue : INT := 0; // Configurable initial value for the counter
END_VAR

// Method to initialize the counter with the initial value
METHOD Initialize(this : REFERENCE TO FB_Counter)
BEGIN
    this.CurrentCount := this.InitialValue;
    this.Overflow := FALSE;
    this.Underflow := FALSE;
END_METHOD

// Main method to execute the counter logic
METHOD Execute(this : REFERENCE TO FB_Counter) : BOOL
BEGIN
    // Reset the counter if the reset flag is set
    IF this.Reset THEN
        this.Initialize();
        RETURN TRUE;
    END_IF;

    // Check if the counter is enabled
    IF this.Enable THEN
        // Clear overflow and underflow flags
        this.Overflow := FALSE;
        this.Underflow := FALSE;

        // Increment or decrement the counter based on the direction flag
        IF this.CountUp THEN
            // Check for potential overflow
            IF this.CurrentCount + this.StepSize > this.MaxValue THEN
                this.Overflow := TRUE;
            ELSE
                this.CurrentCount := this.CurrentCount + this.StepSize;
            END_IF;
        ELSIF NOT this.CountUp THEN
            // Check for potential underflow
            IF this.CurrentCount - this.StepSize < this.MinValue THEN
                this.Underflow := TRUE;
            ELSE
                this.CurrentCount := this.CurrentCount - this.StepSize;
            END_IF;
        END_IF;
    END_IF;

    RETURN TRUE;
END_METHOD

// Main execution method
METHOD MAIN(this : REFERENCE TO FB_Counter) : BOOL
BEGIN
    RETURN this.Execute();
END_METHOD

PROGRAM MainProgram
VAR
    CounterFB : FB_Counter;
    CurrentCount : INT;
    Overflow : BOOL;
    Underflow : BOOL;
END_VAR

// Configure the counter with initial values
CounterFB.InitialValue := 0;
CounterFB.StepSize := 1;
CounterFB.MaxValue := 100;
CounterFB.MinValue := 0;

// Simulate the counter operation
CounterFB(Enable := TRUE, CountUp := TRUE, Reset := FALSE); // Start counting up
CurrentCount := CounterFB.CurrentCount;
Overflow := CounterFB.Overflow;
Underflow := CounterFB.Underflow;

// Output results
IF Overflow THEN
    // Handle overflow condition
ELSEIF Underflow THEN
    // Handle underflow condition
ELSE
    // Use the current count value
END_IF;

// Reset the counter
CounterFB(Enable := FALSE, CountUp := TRUE, Reset := TRUE);
