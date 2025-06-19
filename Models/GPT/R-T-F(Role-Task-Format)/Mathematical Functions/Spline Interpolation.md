FUNCTION_BLOCK SplineInterpolator
VAR_INPUT
    X         : ARRAY[1..10] OF REAL; // Known x-values
    A, B, C, D: ARRAY[1..9] OF REAL;  // Spline coefficients for each interval
    TargetX   : REAL;                 // Target x to interpolate
END_VAR

VAR_OUTPUT
    InterpolatedY : REAL;            // Resulting interpolated y value
END_VAR

VAR
    i : INT;                         // Loop/index variable
    FoundInterval : BOOL;            // Flag to break loop
    DX : REAL;                       // Distance from X[i] to TargetX
END_VAR

// --------- Interval Search ---------
FoundInterval := FALSE;
FOR i := 1 TO 9 DO
    IF (TargetX >= X[i]) AND (TargetX <= X[i+1]) THEN
        FoundInterval := TRUE;
        EXIT;
    END_IF
END_FOR

// --------- Cubic Polynomial Evaluation ---------
IF FoundInterval THEN
    DX := TargetX - X[i];
    InterpolatedY := A[i] + B[i] * DX + C[i] * DX * DX + D[i] * DX * DX * DX;
ELSE
    // Fallback if out of bounds: clamp to edge or return 0
    InterpolatedY := 0.0;
END_IF
