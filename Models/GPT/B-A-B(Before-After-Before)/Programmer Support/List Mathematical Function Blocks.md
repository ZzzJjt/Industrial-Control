// ─────────────────────────────
// 📐 Trigonometric Functions
// ─────────────────────────────

(* Calculates sine of input in radians *)
FUNCTION_BLOCK FB_SIN
VAR_INPUT
    x : REAL; // Angle in radians
END_VAR
VAR_OUTPUT
    y : REAL; // sin(x)
END_VAR
END_FUNCTION_BLOCK

(* Calculates cosine of input in radians *)
FUNCTION_BLOCK FB_COS
VAR_INPUT
    x : REAL;
END_VAR
VAR_OUTPUT
    y : REAL;
END_VAR
END_FUNCTION_BLOCK

(* Calculates arctangent of y/x considering quadrants *)
FUNCTION_BLOCK FB_ATAN2
VAR_INPUT
    y : REAL;
    x : REAL;
END_VAR
VAR_OUTPUT
    angle : REAL; // atan2(y, x)
END_VAR
END_FUNCTION_BLOCK

// ─────────────────────────────
// 📊 Statistical Functions
// ─────────────────────────────

(* Computes mean of an array *)
FUNCTION_BLOCK FB_MEAN
VAR_INPUT
    data : ARRAY[1..100] OF REAL;
END_VAR
VAR_OUTPUT
    avg : REAL;
END_VAR
END_FUNCTION_BLOCK

(* Computes standard deviation of dataset *)
FUNCTION_BLOCK FB_STD_DEV
VAR_INPUT
    data : ARRAY[1..100] OF REAL;
END_VAR
VAR_OUTPUT
    std_dev : REAL;
END_VAR
END_FUNCTION_BLOCK

// ─────────────────────────────
// ➗ Algebraic & Rounding
// ─────────────────────────────

(* Basic add operation *)
FUNCTION_BLOCK FB_ADD
VAR_INPUT
    a, b : REAL;
END_VAR
VAR_OUTPUT
    result : REAL; // a + b
END_VAR
END_FUNCTION_BLOCK

(* Rounds a number to the nearest integer *)
FUNCTION_BLOCK FB_ROUND
VAR_INPUT
    x : REAL;
END_VAR
VAR_OUTPUT
    y : DINT; // rounded result
END_VAR
END_FUNCTION_BLOCK

// ─────────────────────────────
// 🔢 Number Conversion
// ─────────────────────────────

(* Converts INT to REAL *)
FUNCTION_BLOCK FB_INT_TO_REAL
VAR_INPUT
    i : INT;
END_VAR
VAR_OUTPUT
    r : REAL;
END_VAR
END_FUNCTION_BLOCK

(* Converts REAL to INT *)
FUNCTION_BLOCK FB_REAL_TO_INT
VAR_INPUT
    r : REAL;
END_VAR
VAR_OUTPUT
    i : INT;
END_VAR
END_FUNCTION_BLOCK

// ─────────────────────────────
// 🧮 Logarithmic and Exponential
// ─────────────────────────────

(* Calculates exponential of input *)
FUNCTION_BLOCK FB_EXP
VAR_INPUT
    x : REAL;
END_VAR
VAR_OUTPUT
    y : REAL; // e^x
END_VAR
END_FUNCTION_BLOCK

(* Calculates natural logarithm *)
FUNCTION_BLOCK FB_LN
VAR_INPUT
    x : REAL;
END_VAR
VAR_OUTPUT
    y : REAL; // ln(x)
END_VAR
END_FUNCTION_BLOCK

// ─────────────────────────────
// 📦 Array Utilities
// ─────────────────────────────

(* Finds maximum in array *)
FUNCTION_BLOCK FB_ARRAY_MAX
VAR_INPUT
    data : ARRAY[1..100] OF REAL;
END_VAR
VAR_OUTPUT
    max_value : REAL;
END_VAR
END_FUNCTION_BLOCK
