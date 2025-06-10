FUNCTION_BLOCK NaturalLog
VAR_INPUT
    X : REAL; // Input value for natural logarithm
END_VAR

VAR_OUTPUT
    LnX   : REAL; // Output: ln(X)
    Error : BOOL; // TRUE if input is invalid (X <= 0)
END_VAR

// ---------------- Logic ----------------
IF X > 0.0 THEN
    // Valid input: compute natural logarithm using built-in LN() function
    LnX := LN(X); // ln(x) is defined only for x > 0
    Error := FALSE;
ELSE
    // Invalid input: natural logarithm is undefined for X <= 0
    LnX := 0.0;   // Return default (could also be -1.0 or NaN, but 0.0 is PLC-safe)
    Error := TRUE;
END_IF;
