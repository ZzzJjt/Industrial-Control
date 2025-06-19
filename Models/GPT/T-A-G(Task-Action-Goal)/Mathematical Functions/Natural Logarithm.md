FUNCTION_BLOCK NaturalLog
VAR_INPUT
    X : REAL; // Input value for natural logarithm
END_VAR
VAR_OUTPUT
    LnX : REAL;   // Computed natural logarithm ln(X)
    Error : BOOL; // TRUE if input X is invalid (<= 0)
END_VAR
VAR
    // None required for this simple block
END_VAR

// Calculate natural logarithm if input is valid
IF X > 0.0 THEN
    LnX := LN(X);      // Use built-in logarithm function
    Error := FALSE;    // Input is valid
ELSE
    LnX := 0.0;        // Set output to 0 for invalid input
    Error := TRUE;     // Flag error condition
END_IF;
