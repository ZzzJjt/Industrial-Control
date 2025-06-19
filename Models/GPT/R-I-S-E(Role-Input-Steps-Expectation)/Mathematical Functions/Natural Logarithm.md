FUNCTION_BLOCK NaturalLog
VAR_INPUT
    X : REAL; // Input value for which ln(X) is calculated
END_VAR
VAR_OUTPUT
    LnX   : REAL; // Natural logarithm of X
    Error : BOOL; // TRUE if X is out of domain (X ≤ 0)
END_VAR

// Compute natural logarithm only for valid domain
IF X > 0.0 THEN
    LnX := LN(X);     // Use built-in logarithm function
    Error := FALSE;   // No error, input is valid
ELSE
    LnX := -9999.0;   // Default error value (can be customized)
    Error := TRUE;    // Error flag raised for invalid input
END_IF;
