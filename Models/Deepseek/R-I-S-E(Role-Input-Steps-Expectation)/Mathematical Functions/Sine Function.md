FUNCTION_BLOCK FB_SineFunction
VAR_INPUT
    AngleRad : REAL; // Input angle in radians
END_VAR

VAR_OUTPUT
    SineValue : REAL; // The calculated sine of the angle
END_VAR

VAR
    normalizedAngle : REAL; // Normalized angle within [-π, π]
    x, x2, x3, x5, x7 : REAL; // Intermediate terms for Taylor series
    twoPi : REAL := 6.283185307179586; // Constant for normalization
    pi : REAL := 3.141592653589793; // Pi constant
END_VAR

// Normalize angle to be within [-π, π] for better approximation accuracy
normalizedAngle := AngleRad MOD twoPi;
IF normalizedAngle > pi THEN
    normalizedAngle := normalizedAngle - twoPi;
ELSIF normalizedAngle < -pi THEN
    normalizedAngle := normalizedAngle + twoPi;
END_IF;

// Option 1: Use built-in SIN() function if supported by the platform
SineValue := SIN(normalizedAngle);

// Option 2: Fallback to Taylor series approximation if no built-in SIN()
// Uncomment the following section if necessary
(*
x := normalizedAngle;
x2 := x * x;
x3 := x2 * x;
x5 := x2 * x3;
x7 := x2 * x5;

// Taylor series approximation around 0: x - x^3/3! + x^5/5! - x^7/7!
SineValue := x - (x3 / 6.0) + (x5 / 120.0) - (x7 / 5040.0);
*)

// Note: The Taylor series works best near 0, which is why we normalize.
// However, it may still have limited accuracy for angles far from 0.
