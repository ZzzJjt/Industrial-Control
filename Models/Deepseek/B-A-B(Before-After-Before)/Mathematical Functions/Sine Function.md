FUNCTION_BLOCK SineFunction
VAR_INPUT 
    AngleRad : REAL; // Input angle in radians
    UseApproximation : BOOL := FALSE; // Optional flag to force approximation
END_VAR

VAR_OUTPUT 
    SineValue : REAL; // Output sine value
END_VAR

// Variables used for Taylor series calculation
VAR 
    x, x2, x3, x5, x7 : REAL;
END_VAR

IF NOT UseApproximation THEN
    // Attempt to use the built-in SIN function if not forcing approximation
    SineValue := SIN(AngleRad);
ELSE
    // Normalize the angle to within [-π, π] to improve accuracy of approximation
    WHILE AngleRad > PI DO AngleRad := AngleRad - (2 * PI); END_WHILE;
    WHILE AngleRad < -PI DO AngleRad := AngleRad + (2 * PI); END_WHILE;

    x := AngleRad;
    x2 := x * x;
    x3 := x2 * x;
    x5 := x3 * x2;
    x7 := x5 * x2;
    
    // Taylor series approximation around 0: sin(x) ≈ x - x^3/3! + x^5/5! - x^7/7!
    SineValue := x - (x3 / 6.0) + (x5 / 120.0) - (x7 / 5040.0);
END_IF;

