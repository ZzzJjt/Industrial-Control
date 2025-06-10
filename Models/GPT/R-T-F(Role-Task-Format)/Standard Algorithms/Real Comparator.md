FUNCTION_BLOCK FB_RealComparator
VAR_INPUT
    Input1    : REAL; // First real number to compare
    Input2    : REAL; // Second real number to compare
    Precision : INT;  // Number of decimal places (e.g., 3 → 0.001)
    Enable    : BOOL; // Enable execution
END_VAR

VAR_OUTPUT
    Equal     : BOOL := FALSE; // TRUE if Input1 ≈ Input2
    Greater   : BOOL := FALSE; // TRUE if Input1 > Input2
    Less      : BOOL := FALSE; // TRUE if Input1 < Input2
    Error     : BOOL := FALSE; // TRUE if Precision invalid or disabled
END_VAR

VAR
    ScaleFactor   : REAL; // 10^Precision
    Scaled1       : DINT; // Rounded(Input1 * Scale)
    Scaled2       : DINT; // Rounded(Input2 * Scale)
    ValidPrecision: BOOL;
END_VAR

// === Initialization ===
Equal := FALSE;
Greater := FALSE;
Less := FALSE;
Error := FALSE;

// === Validation ===
ValidPrecision := (Precision >= 0) AND (Precision <= 9); // Restrict to practical range

IF NOT Enable OR NOT ValidPrecision THEN
    Error := TRUE;
ELSE
    // === Scale calculation ===
    ScaleFactor := POWER(10.0, REAL(Precision));

    // === Integer scaling and rounding ===
    Scaled1 := ROUND(Input1 * ScaleFactor);
    Scaled2 := ROUND(Input2 * ScaleFactor);

    // === Comparison logic ===
    IF Scaled1 = Scaled2 THEN
        Equal := TRUE;
    ELSIF Scaled1 > Scaled2 THEN
        Greater := TRUE;
    ELSE
        Less := TRUE;
    END_IF
END_IF
