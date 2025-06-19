FUNCTION_BLOCK NaturalLog
VAR_INPUT
    X : REAL; // Input value for which to compute ln(x)
END_VAR

VAR_OUTPUT
    LnX  : REAL; // Output: natural logarithm of X
    Error: BOOL; // Output: TRUE if input is invalid
END_VAR

// Step 1: Check domain validity (ln(x) is undefined for x <= 0)
IF X > 0.0 THEN
    LnX := LN(X);     // Use built-in natural logarithm function
    Error := FALSE;   // Valid input
ELSE
    LnX := 0.0;       // Default result for invalid input
    Error := TRUE;    // Signal domain error
END_IF;
