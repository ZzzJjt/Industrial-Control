FUNCTION_BLOCK SplineInterpolator
VAR_INPUT
    TargetX : REAL; // The X value to interpolate at
    NumPoints : INT; // Number of known points (N)
    X : ARRAY[1..10] OF REAL; // Known X values
    A : ARRAY[1..10] OF REAL; // Precomputed spline a_i (same as Y[i])
    B : ARRAY[1..10] OF REAL; // Precomputed spline b_i
    C : ARRAY[1..10] OF REAL; // Precomputed spline c_i
    D : ARRAY[1..10] OF REAL; // Precomputed spline d_i
END_VAR

VAR_OUTPUT
    InterpolatedY : REAL;
END_VAR

VAR
    i : INT;
    dx : REAL;
    found : BOOL;
END_VAR

// --- Find the interval such that X[i] <= TargetX < X[i+1] ---
found := FALSE;
FOR i := 1 TO NumPoints - 1 DO
    IF (TargetX >= X[i]) AND (TargetX < X[i+1]) THEN
        found := TRUE;
        EXIT;
    END_IF
END_FOR

// Handle case when TargetX == X[NumPoints]
IF NOT found AND (TargetX = X[NumPoints]) THEN
    i := NumPoints - 1;
    found := TRUE;
END_IF

// --- Evaluate spline polynomial if valid interval found ---
IF found THEN
    dx := TargetX - X[i];
    InterpolatedY := A[i] + B[i]*dx + C[i]*dx*dx + D[i]*dx*dx*dx;
ELSE
    InterpolatedY := 0.0; // Default fallback for out-of-bounds
END_IF
