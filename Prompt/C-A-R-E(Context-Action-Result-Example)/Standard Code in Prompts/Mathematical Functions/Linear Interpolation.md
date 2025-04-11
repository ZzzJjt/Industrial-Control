```
FUNCTION_BLOCK LinearInterpolation
VAR_INPUT
    x : REAL; // The x value at which to interpolate
    x1 : REAL; // The x-coordinate of the first point
    y1 : REAL; // The y-coordinate of the first point
    x2 : REAL; // The x-coordinate of the second point
    y2 : REAL; // The y-coordinate of the second point
VAR_OUTPUT
    y : REAL; // The interpolated y value
END_VAR

// Ensure the points are ordered such that x1 <= x <= x2
IF x < x1 THEN
    y := y1;
ELSIF x > x2 THEN
    y := y2;
ELSE
    // Linear interpolation formula: y = y1 + (y2 - y1) * (x - x1) / (x2 - x1)
    y := y1 + ((y2 - y1) * (x - x1)) / (x2 - x1);
END_IF;
END_FUNCTION_BLOCK
```
