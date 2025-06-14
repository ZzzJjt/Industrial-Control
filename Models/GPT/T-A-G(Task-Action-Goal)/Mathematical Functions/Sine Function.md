FUNCTION_BLOCK SineFunction
VAR_INPUT
    AngleRad : REAL; // Input angle in radians
END_VAR
VAR_OUTPUT
    SineValue : REAL; // Output sine of the angle
END_VAR

// --- Calculation using built-in SIN() function ---
// The input must be in radians, not degrees
// Typical range: -PI to +PI for best accuracy, but SIN() handles larger values too

SineValue := SIN(AngleRad);

WHILE AngleRad > 2.0 * 3.14159 DO
    AngleRad := AngleRad - 2.0 * 3.14159;
END_WHILE;
WHILE AngleRad < -2.0 * 3.14159 DO
    AngleRad := AngleRad + 2.0 * 3.14159;
END_WHILE;
