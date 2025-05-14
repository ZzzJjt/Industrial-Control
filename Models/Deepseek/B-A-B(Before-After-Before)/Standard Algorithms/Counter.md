Below is a **self-contained IEC 61131-3 Structured Text function block** designed for implementing a flexible digital counter in PLC-based industrial automation systems. This function block supports up/down counting modes, configurable initial values, customizable step sizes, and includes overflow and underflow protection.

---

# 🔢 Flexible Digital Counter  
## ✅ IEC 61131-3 Structured Text | Modular Function Block

---

### 📋 Interface Definition

```pascal
FUNCTION_BLOCK FB_FlexibleCounter
VAR_INPUT
    Enable     : BOOL; // Enables the counter operation
    CountUp    : BOOL; // TRUE for up-counting, FALSE for down-counting
    StepSize   : INT;  // Increment/Decrement step size
    InitValue  : INT;  // Initial value of the counter
    Reset      : BOOL; // Resets the counter to InitValue
    MaxValue   : INT;  // Maximum limit for the counter (inclusive)
    MinValue   : INT;  // Minimum limit for the counter (inclusive)
END_VAR

VAR_OUTPUT
    CurrentValue : INT; // The current value of the counter
    AtMax        : BOOL; // TRUE if at maximum limit
    AtMin        : BOOL; // TRUE if at minimum limit
END_VAR

VAR
    internalValue : INT; // Internal storage for the counter value
END_VAR
```

### 🚀 Implementation Logic

```pascal
// Initialize or reset the counter
IF Reset THEN
    internalValue := InitValue;
ELSIF Enable THEN
    // Determine direction based on CountUp flag
    IF CountUp THEN
        // Check for overflow before incrementing
        IF internalValue < MaxValue - StepSize THEN
            internalValue := internalValue + StepSize;
        ELSE
            internalValue := MaxValue;
        END_IF;
    ELSE
        // Check for underflow before decrementing
        IF internalValue > MinValue + StepSize THEN
            internalValue := internalValue - StepSize;
        ELSE
            internalValue := MinValue;
        END_IF;
    END_IF;
END_IF;

// Update outputs
CurrentValue := internalValue;
AtMax := internalValue >= MaxValue;
AtMin := internalValue <= MinValue;
```

### 🛠 Usage Example

Here's how you might use `FB_FlexibleCounter` within your PLC program:

```pascal
PROGRAM MainProgram
VAR
    MyFlexibleCounter : FB_FlexibleCounter;
    CounterEnable     : BOOL := TRUE; // Enable counter operation
    UpCount           : BOOL := TRUE; // Direction of counting
    Step              : INT := 5;     // Step size
    StartValue        : INT := 0;     // Initial value
    ResetCounter      : BOOL := FALSE; // Reset condition
    UpperLimit        : INT := 100;   // Max value
    LowerLimit        : INT := -20;   // Min value
    CurrentCount      : INT;          // Output: current count value
    IsAtMax           : BOOL;         // Output: at max value flag
    IsAtMin           : BOOL;         // Output: at min value flag
END_VAR

// Call the flexible counter function block
MyFlexibleCounter(
    Enable := CounterEnable,
    CountUp := UpCount,
    StepSize := Step,
    InitValue := StartValue,
    Reset := ResetCounter,
    MaxValue := UpperLimit,
    MinValue := LowerLimit
);

// Retrieve output values from the function block
CurrentCount := MyFlexibleCounter.CurrentValue;
IsAtMax := MyFlexibleCounter.AtMax;
IsAtMin := MyFlexibleCounter.AtMin;
```

### 📌 Key Features & Benefits

| Feature | Description |
|--------|-------------|
| **Configurable Direction** | Supports both up and down counting via `CountUp` input |
| **Custom Step Size** | Adjustable increments or decrements using `StepSize` |
| **Overflow/Underflow Protection** | Automatically clamps values between `MinValue` and `MaxValue` |
| **Initialization & Reset** | Can be initialized or reset to a specified value (`InitValue`) |
| **Status Flags** | Provides feedback through `AtMax` and `AtMin` flags when limits are reached |
| **Modular Design** | Easy to integrate and reuse across different projects |

### ⚠️ Notes on Practical Use

When deploying this function block in your system, consider the following:

- Ensure that `StepSize`, `MaxValue`, and `MinValue` are configured appropriately for your application to avoid unexpected behavior.
- The `Reset` input should be used carefully to prevent unintended resets during normal operation.
- For applications requiring more complex conditions (e.g., counting based on external events), additional logic can be added around the function block call.

This function block serves as a robust foundation for various counting needs in industrial control systems, providing flexibility and safety in managing digital counts.
