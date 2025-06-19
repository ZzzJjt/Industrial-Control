Certainly! Below is a self-contained function block in IEC 61131-3 Structured Text that implements a versatile digital counter. This function block supports up and down counting modes, configurable initial values and step sizes, overflow and underflow protection, and an optional reset function.

### Versatile Digital Counter Function Block

```st
[<title="VersatileDigitalCounter">]
FUNCTION_BLOCK DigitalCounterFB
VAR_INPUT
    Enable : BOOL;          // Enable signal to start the counter
    Reset : BOOL;           // Reset signal to reinitialize the counter
    CountUp : BOOL;         // Direction control: TRUE for up-counting, FALSE for down-counting
    StepSize : INT := 1;     // Step size for each count increment/decrement
    InitValue : INT := 0;    // Initial value of the counter
    MaxValue : INT := 1000;  // Maximum value before overflow
    MinValue : INT := 0;     // Minimum value before underflow
END_VAR

VAR_OUTPUT
    CurrentValue : INT := 0; // Current value of the counter
    AtMax : BOOL := FALSE;   // Flag indicating if the counter has reached the maximum value
    AtMin : BOOL := FALSE;   // Flag indicating if the counter has reached the minimum value
END_VAR

METHOD Execute : VOID
IF Reset THEN
    CurrentValue := InitValue;
    AtMax := FALSE;
    AtMin := FALSE;
ELSIF Enable THEN
    IF CountUp THEN
        CurrentValue := CurrentValue + StepSize;
        IF CurrentValue >= MaxValue THEN
            CurrentValue := MaxValue;
            AtMax := TRUE;
        ELSE
            AtMax := FALSE;
        END_IF;
    ELSE
        CurrentValue := CurrentValue - StepSize;
        IF CurrentValue <= MinValue THEN
            CurrentValue := MinValue;
            AtMin := TRUE;
        ELSE
            AtMin := FALSE;
        END_IF;
    END_IF;
END_IF;
END_METHOD
```

### Explanation of the Code

#### Inputs
- **Enable**: A boolean flag to enable the counter.
- **Reset**: A boolean flag to reset the counter to its initial value.
- **CountUp**: A boolean flag to determine the direction of counting (`TRUE` for up-counting, `FALSE` for down-counting).
- **StepSize**: An integer representing the step size for each count increment or decrement (default is 1).
- **InitValue**: An integer representing the initial value of the counter (default is 0).
- **MaxValue**: An integer representing the maximum value before overflow (default is 1000).
- **MinValue**: An integer representing the minimum value before underflow (default is 0).

#### Outputs
- **CurrentValue**: The current value of the counter.
- **AtMax**: A boolean flag indicating whether the counter has reached the maximum value.
- **AtMin**: A boolean flag indicating whether the counter has reached the minimum value.

#### Method: Execute
- **Reset Logic**: If the `Reset` input is `TRUE`, the `CurrentValue` is set to `InitValue`, and both `AtMax` and `AtMin` flags are cleared.
- **Counting Logic**:
  - If `Enable` is `TRUE`:
    - If `CountUp` is `TRUE`, the `CurrentValue` is incremented by `StepSize`.
      - If `CurrentValue` exceeds `MaxValue`, it is clamped to `MaxValue`, and `AtMax` is set to `TRUE`.
    - If `CountUp` is `FALSE`, the `CurrentValue` is decremented by `StepSize`.
      - If `CurrentValue` falls below `MinValue`, it is clamped to `MinValue`, and `AtMin` is set to `TRUE`.

### Usage Example

To use this function block in a PLC program, you would instantiate it and call its `Execute` method:

```st
PROGRAM MainProgram
VAR
    MyCounter : DigitalCounterFB;
BEGIN
    // Configure the counter
    MyCounter.Enable := TRUE;
    MyCounter.Reset := FALSE;
    MyCounter.CountUp := TRUE;
    MyCounter.StepSize := 5;
    MyCounter.InitValue := 0;
    MyCounter.MaxValue := 100;
    MyCounter.MinValue := 0;

    // Execute the counter logic
    MyCounter.Execute();

    // Output the current value and status
    IF MyCounter.AtMax THEN
        // Counter has reached the maximum value
    ELSIF MyCounter.AtMin THEN
        // Counter has reached the minimum value
    ELSE
        // Counter is within range
    END_IF;

    // Optionally, reset the counter after some condition
    MyCounter.Reset := TRUE;
    MyCounter.Execute();
END_PROGRAM
```

This function block is modular, reliable, and reusable, making it suitable for various applications in industrial automation systems where flexible and safe counting logic is required.
