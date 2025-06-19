FUNCTION_BLOCK NaturalLog
VAR_INPUT
    X : REAL; // Input value to compute ln(X)
END_VAR

VAR_OUTPUT
    LnX : REAL; // Result of ln(X)
    Error : BOOL; // Set to TRUE if X ≤ 0, indicating an invalid input
END_VAR

METHOD Execute : BOOL
BEGIN
    // Check if the input value X is greater than 0
    IF X > 0.0 THEN
        // Compute the natural logarithm using the built-in LN() function
        LnX := LN(X);
        Error := FALSE; // No error occurred
    ELSE
        // Set the result to 0.0 and raise an error for invalid input (X ≤ 0)
        LnX := 0.0;
        Error := TRUE; // Indicate invalid input
    END_IF;

    RETURN TRUE;
END_METHOD

// Main execution loop
Execute();



