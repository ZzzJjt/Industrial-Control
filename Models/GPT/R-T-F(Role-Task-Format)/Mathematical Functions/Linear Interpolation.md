FUNCTION_BLOCK LinearInterpolation
VAR_INPUT
    X1 : REAL;   // First known X value
    Y1 : REAL;   // First known Y value
    X2 : REAL;   // Second known X value
    Y2 : REAL;   // Second known Y value
    X  : REAL;   // Target input value to interpolate at
END_VAR
VAR_OUTPUT
    Y : REAL;    // Interpolated output value
END_VAR

// ----------------------
// Linear interpolation formula:
// Y = Y1 + ((X - X1) * (Y2 - Y1)) / (X2 - X1)
// Guard against division by zero (X1 = X2)
// ----------------------
IF X2 <> X1 THEN
    // Normal interpolation
    Y := Y1 + ((X - X1) * (Y2 - Y1)) / (X2 - X1);
ELSE
    // Avoid divide-by-zero: fallback to Y1 or error handling
    Y := Y1; // You may also set Y := 0.0 or raise an alarm
END_IF;
