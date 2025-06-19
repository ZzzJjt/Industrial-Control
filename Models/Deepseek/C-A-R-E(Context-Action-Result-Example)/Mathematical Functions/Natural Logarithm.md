FUNCTION_BLOCK NaturalLog
VAR_INPUT
    X : REAL; // Input value for which to calculate the natural logarithm
END_VAR

VAR_OUTPUT
    LnX : REAL; // Output: natural logarithm of X
    Error : BOOL; // Error flag: TRUE if input is zero or negative
END_VAR

// Initialize outputs
LnX := 0.0;
Error := FALSE;

// Check if the input is valid (greater than 0)
IF X > 0.0 THEN
    // Calculate the natural logarithm using the built-in LN function
    LnX := LN(X);
    Error := FALSE;
ELSE
    // Set output to 0.0 and raise the error flag for invalid input
    LnX := 0.0;
    Error := TRUE;
END_IF;

// Inline comments explaining the logic:
// The natural logarithm ln(x) is undefined for x ≤ 0.
// The LN() function is used to compute the natural logarithm for valid inputs.
// If the input is zero or negative, the function sets the output to 0.0 and raises the error flag.
// This ensures safe behavior and deterministic execution, preventing runtime faults.



