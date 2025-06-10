FUNCTION_BLOCK SineFunction
VAR_INPUT
    AngleRad : REAL; // Input angle in radians
END_VAR

VAR_OUTPUT
    SineValue : REAL; // Computed sine of the input angle
END_VAR

VAR
    x   : REAL; // Angle (may be normalized to improve precision)
    x2  : REAL; // x^2
    x3  : REAL; // x^3
    x5  : REAL; // x^5
    x7  : REAL; // x^7
END_VAR

// --------- Normalize input angle to range [-PI, PI] for stability ---------
// Optional: uncomment if needed for large input ranges
// WHILE AngleRad > 3.14159265 DO
//     AngleRad := AngleRad - 2.0 * 3.14159265;
// END_WHILE;
// WHILE AngleRad < -3.14159265 DO
//     AngleRad := AngleRad + 2.0 * 3.14159265;
// END_WHILE;

// --------- Compute sine using Taylor Series approximation ---------
// sin(x) ≈ x - x^3/3! + x^5/5! - x^7/7!

x := AngleRad;
x2 := x * x;
x3 := x2 * x;
x5 := x3 * x2;
x7 := x5 * x2;

SineValue := x 
           - (x3 / 6.0)       // 3! = 6
           + (x5 / 120.0)     // 5! = 120
           - (x7 / 5040.0);   // 7! = 5040
