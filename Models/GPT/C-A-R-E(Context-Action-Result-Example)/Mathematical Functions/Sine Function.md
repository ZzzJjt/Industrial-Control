FUNCTION_BLOCK SineFunction
VAR_INPUT
    AngleRad : REAL; // Input angle in radians
END_VAR

VAR_OUTPUT
    SineValue : REAL; // Output sine value of the input angle
END_VAR

VAR
    AngleNorm  : REAL; // Normalized angle in range [-PI, PI]
    Term       : REAL; // Current term in Taylor series
    Numerator  : REAL;
    Denominator: REAL;
    I          : INT;
END_VAR

// === METHOD 1: Built-in SIN() function available ===
SineValue := SIN(AngleRad);

// === METHOD 2: (Optional) Taylor Series Approximation ===
// Uncomment the following section if SIN() is unavailable on your PLC
(*
AngleNorm := AngleRad;

// Optional: Normalize to [-PI, PI] to improve approximation accuracy
WHILE AngleNorm > 3.14159265359 DO
    AngleNorm := AngleNorm - 2.0 * 3.14159265359;
END_WHILE;
WHILE AngleNorm < -3.14159265359 DO
    AngleNorm := AngleNorm + 2.0 * 3.14159265359;
END_WHILE;

// Taylor series expansion: sin(x) ≈ x - x³/3! + x⁵/5! - x⁷/7! + ...
SineValue := 0.0;
Term := AngleNorm;
Numerator := AngleNorm;
Denominator := 1.0;

FOR I := 1 TO 5 DO
    SineValue := SineValue + Term;
    Numerator := -Numerator * AngleNorm * AngleNorm;
    Denominator := Denominator * (2 * I) * (2 * I + 1);
    Term := Numerator / Denominator;
END_FOR;
*)
