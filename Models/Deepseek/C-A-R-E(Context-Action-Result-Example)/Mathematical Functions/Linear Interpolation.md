FUNCTION_BLOCK LinearInterpolation
VAR_INPUT
    X1 : REAL; // X-coordinate of the first known point
    Y1 : REAL; // Y-coordinate of the first known point
    X2 : REAL; // X-coordinate of the second known point
    Y2 : REAL; // Y-coordinate of the second known point
    X : REAL;  // Query point for which Y is interpolated
END_VAR

VAR_OUTPUT
    Y : REAL; // Interpolated Y value
END_VAR

VAR
    DeltaX : REAL; // Difference between X2 and X1
    DeltaY : REAL; // Difference between Y2 and Y1
    Slope : REAL;  // Slope of the line segment
    Intercept : REAL; // Intercept of the line segment
END_VAR

// Calculate differences
DeltaX := X2 - X1;
DeltaY := Y2 - Y1;

// Check for zero denominator to avoid division error
IF DeltaX <> 0.0 THEN
    // Calculate slope and intercept
    Slope := DeltaY / DeltaX;
    Intercept := Y1 - (Slope * X1);
    
    // Perform linear interpolation
    Y := Slope * X + Intercept;
ELSE
    // Handle the case where X1 = X2 (vertical line)
    // Default to Y1 since no unique Y can be determined
    Y := Y1;
END_IF;



