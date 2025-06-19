Here is a **self-contained IEC 61131-3 Structured Text function block** that performs a **Fahrenheit-to-Celsius temperature conversion**, including input validation and safe output handling. This function block is designed to be reusable, robust, and suitable for use in PLC-based industrial automation systems involving HVAC, process control, or sensor integration.

---

## 🌡️ FUNCTION_BLOCK: `FB_FtoC_Converter`

### ✅ Purpose

Converts a temperature value from Fahrenheit (REAL) to Celsius (REAL), ensuring physical validity by checking against absolute zero (-459.67°F). The block includes:

- Input validation
- Accurate floating-point arithmetic
- Clear status flagging (`ValidInput`)
- Safe default behavior on invalid input

---

### 🧱 Interface Definition

```pascal
FUNCTION_BLOCK FB_FtoC_Converter
VAR_INPUT
    Fahrenheit : REAL; // Input temperature in degrees Fahrenheit
END_VAR

VAR_OUTPUT
    Celsius     : REAL; // Output temperature in degrees Celsius
    ValidInput  : BOOL; // TRUE if input is physically valid
END_VAR
```

---

### 🚀 Implementation Logic

```pascal
// Absolute zero in degrees Fahrenheit
CONST
    ABS_ZERO_FAHRENHEIT : REAL := -459.67;
END_CONST

// Conversion logic
IF Fahrenheit >= ABS_ZERO_FAHRENHEIT THEN
    // Apply the standard Fahrenheit-to-Celsius formula:
    // C = (F - 32) * (5 / 9)
    Celsius := (Fahrenheit - 32.0) * (5.0 / 9.0);
    ValidInput := TRUE;
ELSE
    // Input is below absolute zero — physically impossible
    Celsius := 0.0;
    ValidInput := FALSE;
END_IF;
```

---

### 🛠 Example Usage in a PLC Program

```pascal
PROGRAM PLC_PRG
VAR
    TempConverter : FB_FtoC_Converter;
    InputF          : REAL := 212.0; // Boiling point of water
    OutputC         : REAL;
    IsValid         : BOOL;
END_VAR

// Call the function block
TempConverter(
    Fahrenheit := InputF,
    Celsius => OutputC,
    ValidInput => IsValid
);

// Output values now available:
// OutputC = 100.0 when InputF = 212.0
// IsValid = TRUE
```

---

### 📌 Key Features & Benefits

| Feature | Description |
|--------|-------------|
| **Physical Validation** | Rejects unphysical inputs like temperatures below absolute zero |
| **Accurate Conversion** | Uses IEEE 754-compliant floating-point math |
| **Safe Defaults** | Sets Celsius to 0 when input is invalid |
| **Modular Design** | Encapsulated logic for reuse across programs |
| **IEC 61131-3 Compliant** | Compatible with CODESYS, TwinCAT, Siemens TIA Portal, Beckhoff TwinCAT, etc. |

---

### ⚠️ Notes on Practical Use

| Consideration | Recommendation |
|--------------|----------------|
| **Floating Point Precision** | Use `LREAL` if supported for higher precision (e.g., in safety-critical applications) |
| **Input Filtering** | Pre-filter noisy analog sensor inputs before passing to this function block |
| **Error Handling** | Extend with error codes or diagnostics if needed |
| **Scan Cycle Impact** | This block is lightweight and scan-cycle-safe |
| **Temperature Display** | Connect `Celsius` output directly to HMI or logging system |

---

### 📥 Optional Extensions

Would you like me to provide versions of this function block that include:

✅ Support for **LREAL** data types for higher precision  
✅ Reverse conversion (**Celsius to Fahrenheit**)  
✅ Integration with **temperature scaling from analog inputs**  
✅ **Error codes** instead of just a boolean flag  
✅ A **test harness** for simulating edge cases (e.g., invalid values)

Let me know how you'd like to expand upon this!
