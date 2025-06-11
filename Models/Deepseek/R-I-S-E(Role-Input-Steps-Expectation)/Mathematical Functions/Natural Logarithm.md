FUNCTION_BLOCK FB_NaturalLog
VAR_INPUT
    X : REAL := 0.0; // Input value to compute ln(X)
END_VAR

VAR_OUTPUT
    LnX : REAL := 0.0; // Natural logarithm of X
    Error : BOOL := FALSE; // TRUE if input is ≤ 0
END_VAR

VAR
    epsilon : REAL := 1.0E-9; // Small positive threshold to avoid numerical issues
END_VAR

// Check if input is within the domain of natural logarithm (X > 0)
IF X > epsilon THEN
    // Valid input: compute natural logarithm using built-in LN() function
    LnX := LN(X);
    Error := FALSE;
ELSE
    // Invalid input: X ≤ 0 has no real logarithm
    LnX := -9999.0; // Placeholder value indicating undefined result
    Error := TRUE;
END_IF;

PROGRAM PLC_PRG
VAR
    LogCalculator: FB_NaturalLog;

    RawInput : REAL := 2.71828; // e ≈ 2.71828
    LogResult : REAL := 0.0;
    LogError : BOOL := FALSE;
END_VAR

// Compute natural log
LogCalculator(
    X := RawInput
);

LogResult := LogCalculator.LnX;
LogError := LogCalculator.Error;
