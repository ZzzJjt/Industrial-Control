FUNCTION_BLOCK NaturalLog
VAR_INPUT 
    X : REAL; // The value for which to compute ln(X), must be greater than 0
END_VAR

VAR_OUTPUT 
    LnX : REAL; // The computed natural logarithm of X
    Error : BOOL; // TRUE if input is invalid (X <= 0)
END_VAR

// Additional variables can be declared here if needed


IF X > 0.0 THEN
    LnX := LN(X); // Use the built-in natural logarithm function where supported
    Error := FALSE;
ELSE
    LnX := 0.0; // Alternatively, could return another specific value
    Error := TRUE; // Indicates that the input is invalid
END_IF;

