FUNCTION_BLOCK SineFunction
VAR_INPUT
    AngleRad : REAL; // Input angle in radians
END_VAR
VAR_OUTPUT
    SineValue : REAL; // Calculated sine of the angle
END_VAR
VAR
    x     : REAL; // Normalized angle [-π, π]
    x2    : REAL;
    x3    : REAL;
    x5    : REAL;
    x7    : REAL;
    PI    : REAL := 3.14159265;
END_VAR

// Normalize the input angle to the range [-π, π] for improved accuracy
x := AngleRad;
WHILE x > PI DO
    x := x - (2.0 * PI);
END_WHILE;
WHILE x < -PI DO
    x := x + (2.0 * PI);
END_WHILE;

// Precompute powers of x
x2 := x * x;
x3 := x2 * x;
x5 := x3 * x2;
x7 := x5 * x2;

// Taylor series approximation for sin(x) around 0 (7th order):
// sin(x) ≈ x - x³/3! + x⁵/5! - x⁷/7!
SineValue := x - (x3 / 6.0) + (x5 / 120.0) - (x7 / 5040.0);
