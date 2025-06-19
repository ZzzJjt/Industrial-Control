FUNCTION_BLOCK NaturalLog
VAR_INPUT
    X : REAL; // Input value for which ln(x) is computed
END_VAR

VAR_OUTPUT
    LnX   : REAL; // Output: natural logarithm of X
    Error : BOOL; // Error flag: TRUE if X <= 0
END_VAR

// Step 1: Validate input
IF X > 0.0 THEN
    // Valid input: use built-in LN function
    LnX := LN(X);
    Error := FALSE;
ELSE
    // Invalid input: ln(x) is undefined for x ≤ 0
    LnX := 0.0; // Safe fallback
    Error := TRUE; // Raise error flag
END_IF;

