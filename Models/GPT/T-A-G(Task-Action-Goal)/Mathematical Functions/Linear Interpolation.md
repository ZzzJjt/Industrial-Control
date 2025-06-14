FUNCTION_BLOCK LinearInterpolation
VAR_INPUT
    X1 : REAL; // First known X value
    Y1 : REAL; // First known Y value
    X2 : REAL; // Second known X value
    Y2 : REAL; // Second known Y value
    X  : REAL; // X value to interpolate at
END_VAR

VAR_OUTPUT
    Y  : REAL; // Interpolated output Y
END_VAR

// Compute linear interpolation: Y = Y1 + ((X - X1) * (Y2 - Y1)) / (X2 - X1)
// Check for division by zero
IF X2 <> X1 THEN
    Y := Y1 + ((X - X1) * (Y2 - Y1)) / (X2 - X1);
ELSE
    Y := Y1; // If X1 = X2, fallback to Y1
END_IF;
