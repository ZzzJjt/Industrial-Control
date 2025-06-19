FUNCTION_BLOCK SplineInterpolator
VAR_INPUT
    X : ARRAY[1..10] OF REAL; // Known x-values (sorted ascending)
    Y : ARRAY[1..10] OF REAL; // Corresponding y-values
    A : ARRAY[1..9] OF REAL;  // Spline coefficient A[i]
    B : ARRAY[1..9] OF REAL;  // Spline coefficient B[i]
    C : ARRAY[1..9] OF REAL;  // Spline coefficient C[i]
    D : ARRAY[1..9] OF REAL;  // Spline coefficient D[i]
    TargetX : REAL;           // The x-value to interpolate
END_VAR
VAR_OUTPUT
    InterpolatedY : REAL;     // Interpolated output value
END_VAR
VAR
    i : INT;
    dx : REAL;
    Found : BOOL := FALSE;
END_VAR

// Loop to find the interval [X[i], X[i+1]] that contains TargetX
FOR i := 1 TO 9 DO
    IF (TargetX >= X[i]) AND (TargetX <= X[i+1]) THEN
        dx := TargetX - X[i];
        InterpolatedY := A[i] + B[i]*dx + C[i]*dx*dx + D[i]*dx*dx*dx;
        Found := TRUE;
        EXIT;
    END_IF;
END_FOR;

// If TargetX is out of bounds, clamp to nearest value (or set error handling)
IF NOT Found THEN
    IF TargetX < X[1] THEN
        InterpolatedY := Y[1];
    ELSIF TargetX > X[10] THEN
        InterpolatedY := Y[10];
    END_IF;
END_IF;
