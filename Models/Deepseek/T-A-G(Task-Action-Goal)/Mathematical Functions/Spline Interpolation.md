FUNCTION_BLOCK SplineInterpolator
VAR_INPUT
    X : ARRAY[1..N] OF REAL; // Known x-values (must be sorted in ascending order)
    Y : ARRAY[1..N] OF REAL; // Corresponding y-values
    TargetX : REAL; // The x-value at which to interpolate
END_VAR

VAR_OUTPUT
    InterpolatedY : REAL; // The result of spline interpolation
END_VAR

VAR
    N : INT := SIZEOF(X); // Number of data points
    A : ARRAY[1..N] OF REAL; // Precomputed coefficients
    B : ARRAY[1..N] OF REAL; // Precomputed coefficients
    C : ARRAY[1..N] OF REAL; // Precomputed coefficients
    D : ARRAY[1..N] OF REAL; // Precomputed coefficients
    Initialized : BOOL := FALSE; // Flag to check if coefficients are initialized
END_VAR

METHOD InitializeCoefficients : BOOL
VAR
    h : ARRAY[1..N-1] OF REAL;
    alpha : ARRAY[1..N-1] OF REAL;
    l : ARRAY[1..N] OF REAL;
    mu : ARRAY[1..N-1] OF REAL;
    z : ARRAY[1..N] OF REAL;
    i : INT;
BEGIN
    // Compute differences between consecutive x-values
    FOR i := 1 TO N-1 DO
        h[i] := X[i+1] - X[i];
    END_FOR;

    // Compute alpha values
    FOR i := 2 TO N-1 DO
        alpha[i] := (3.0 / h[i]) * (Y[i+1] - Y[i]) - (3.0 / h[i-1]) * (Y[i] - Y[i-1]);
    END_FOR;

    // Forward elimination
    l[1] := 1.0;
    mu[1] := 0.0;
    z[1] := 0.0;

    FOR i := 2 TO N-1 DO
        l[i] := 2.0 * (X[i+1] - X[i-1]) - h[i-1] * mu[i-1];
        mu[i] := h[i] / l[i];
        z[i] := (alpha[i] - h[i-1] * z[i-1]) / l[i];
    END_FOR;

    l[N] := 1.0;
    z[N] := 0.0;
    c[N] := 0.0;

    // Backward substitution
    FOR i := N-1 TO 1 BY -1 DO
        c[i] := z[i] - mu[i] * c[i+1];
        b[i] := (Y[i+1] - Y[i]) / h[i] - h[i] * (c[i+1] + 2.0 * c[i]) / 3.0;
        d[i] := (c[i+1] - c[i]) / (3.0 * h[i]);
    END_FOR;

    // Set initial coefficients
    FOR i := 1 TO N DO
        a[i] := Y[i];
    END_FOR;

    Initialized := TRUE;
    RETURN TRUE;
END_METHOD

METHOD Execute : BOOL
VAR
    i : INT;
    j : INT;
    dx : REAL;
BEGIN
    IF NOT Initialized THEN
        InitializeCoefficients();
    END_IF;

    // Binary search to find the correct interval for TargetX
    i := 1;
    j := N;
    WHILE j - i > 1 DO
        INT_TO_INT((i + j) / 2, j);
        IF X[j] > TargetX THEN
            j := j;
        ELSE
            i := j;
        END_IF;
    END_WHILE;

    // Evaluate the spline polynomial for the found interval
    dx := TargetX - X[i];
    InterpolatedY := a[i] + b[i] * dx + c[i] * dx * dx + d[i] * dx * dx * dx;

    RETURN TRUE;
END_METHOD

// Main execution loop
Execute();



