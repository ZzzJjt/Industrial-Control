FUNCTION_BLOCK SineFunction
VAR_INPUT
    AngleRad         : REAL; // Input angle in radians
    UseApproximation : BOOL; // TRUE = use Taylor series, FALSE = use native SIN()
END_VAR

VAR_OUTPUT
    SineValue : REAL; // Output sine value
END_VAR

VAR
    x, x2, x3, x5, x7 : REAL; // Intermediate terms for Taylor expansion
END_VAR

// --- Execution logic ---
IF NOT UseApproximation THEN
    // Use platform's built-in SIN function if available
    SineValue := SIN(AngleRad);
ELSE
    // Use Taylor series expansion for sin(x)
    // sin(x) ≈ x - x^3/3! + x^5/5! - x^7/7!
    // Valid primarily for |x| < π (normalize larger angles for accuracy)
    x := AngleRad;
    x2 := x * x;
    x3 := x2 * x;
    x5 := x3 * x2;
    x7 := x5 * x2;

    SineValue := x - (x3 / 6.0) + (x5 / 120.0) - (x7 / 5040.0);
END_IF;
