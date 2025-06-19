FUNCTION_BLOCK NaturalLog
VAR_INPUT
    X : REAL; // Input value for which to compute the natural logarithm
END_VAR

VAR_OUTPUT
    LnX : REAL; // Result of ln(X)
    Error : BOOL; // Set to TRUE if X <= 0 (invalid input)
END_VAR

METHOD Execute : BOOL
BEGIN
    // Check if the input X is greater than 0
    IF X > 0.0 THEN
        // Compute the natural logarithm using the built-in LN() function
        LnX := LN(X);
        Error := FALSE;
    ELSE
        // Set default fallback value for LnX
        LnX := 0.0;
        // Set error flag to TRUE for invalid input
        Error := TRUE;
    END_IF;

    RETURN TRUE;
END_METHOD

END_FUNCTION_BLOCK
