To address the issue of comparing floating-point values (REAL types) in industrial control systems accurately, we will develop a self-contained function block in IEC 61131-3 Structured Text. This function block will compare two REAL inputs up to a specified number of decimal places, handling floating-point inaccuracies and including error checks.

### 🧱 FUNCTION_BLOCK: `FB_CompareReal`

#### ✅ Purpose

Provide reliable comparison functionality for REAL numbers by rounding them to a specified precision level before comparison. This helps mitigate issues arising from floating-point representation inaccuracies.

---

### 📋 Interface Definition

```pascal
FUNCTION_BLOCK FB_CompareReal
VAR_INPUT
    Input1   : REAL; // First input value
    Input2   : REAL; // Second input value
    Precision: INT;  // Number of decimal places for comparison
    Enable   : BOOL; // Enable comparison operation
END_VAR

VAR_OUTPUT
    Equal    : BOOL; // TRUE if Input1 equals Input2 within precision
    Greater  : BOOL; // TRUE if Input1 is greater than Input2 within precision
    Less     : BOOL; // TRUE if Input1 is less than Input2 within precision
    Error    : BOOL; // TRUE if an error occurred during comparison (e.g., invalid precision)
END_VAR

VAR
    ScaleFactor : REAL; // Scale factor based on Precision
END_VAR
```

### 🚀 Implementation Logic

```pascal
// Initialization or reset
Error := FALSE;
Equal := FALSE;
Greater := FALSE;
Less := FALSE;

// Validate Precision
IF Precision < 0 THEN
    Error := TRUE;
    EXIT;
END_IF;

// Calculate ScaleFactor based on Precision
ScaleFactor := POWER(10, Precision);

// Ensure Enable is true before proceeding with comparison
IF Enable AND NOT Error THEN
    // Normalize both inputs by scaling and rounding
    VAR scaledInput1 : INT := ROUND(Input1 * ScaleFactor);
    VAR scaledInput2 : INT := ROUND(Input2 * ScaleFactor);

    // Compare normalized inputs
    IF scaledInput1 = scaledInput2 THEN
        Equal := TRUE;
    ELSIF scaledInput1 > scaledInput2 THEN
        Greater := TRUE;
    ELSE
        Less := TRUE;
    END_IF;
END_IF;
```

### 🛠 Example Usage in a PLC Program

```pascal
PROGRAM MainProgram
VAR
    MyCompareReal : FB_CompareReal;
    ValueA        : REAL := 3.14159;
    ValueB        : REAL := 3.14158;
    Decimals      : INT := 5;
    CompEnable    : BOOL := TRUE;
    AreEqual      : BOOL;
    IsGreater     : BOOL;
    IsLess        : BOOL;
    ComparisonError: BOOL;
END_VAR

// Call the function block for comparison
MyCompareReal(
    Input1 := ValueA,
    Input2 := ValueB,
    Precision := Decimals,
    Enable := CompEnable,
    Equal => AreEqual,
    Greater => IsGreater,
    Less => IsLess,
    Error => ComparisonError
);

// Use outputs AreEqual, IsGreater, IsLess in your logic as needed
```

### 📌 Key Features & Benefits

| Feature | Description |
|--------|-------------|
| **Precision Control** | Allows users to define how many decimal places are considered significant for comparisons. |
| **Floating Point Handling** | Normalizes inputs through scaling and rounding to avoid common floating point inaccuracies. |
| **Error Checking** | Ensures that operations only proceed when valid inputs are provided, signaling errors otherwise. |
| **Modular Design** | Easy to integrate into larger programs, providing clear and reliable comparison results. |

This function block serves as a robust solution for reliably comparing REAL values in industrial automation applications, ensuring that logic based on these comparisons remains accurate and stable despite inherent limitations of floating-point arithmetic.
