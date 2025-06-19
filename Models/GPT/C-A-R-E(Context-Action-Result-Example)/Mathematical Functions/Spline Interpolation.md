FUNCTION_BLOCK SplineInterpolator
VAR_INPUT
    TargetX : REAL; // Query point
END_VAR

VAR_OUTPUT
    InterpolatedY : REAL; // Output: spline-interpolated value at TargetX
END_VAR

VAR_IN_OUT
    X : ARRAY[1..10] OF REAL; // Known x-values (monotonically increasing)
    Y : ARRAY[1..10] OF REAL; // Known y-values
    A : ARRAY[1..9] OF REAL;  // Coefficient a for each interval
    B : ARRAY[1..9] OF REAL;  // Coefficient b
    C : ARRAY[1..9] OF REAL;  // Coefficient c
    D : ARRAY[1..9] OF REAL;  // Coefficient d
END_VAR

VAR
    i : INT;   // Interval index
    dx : REAL; // Difference (TargetX - X[i])
END_VAR

// === Step 1: Find interval such that X[i] <= TargetX < X[i+1] ===
FOR i := 1 TO 9 DO
    IF (TargetX >= X[i]) AND (TargetX < X[i+1]) THEN
        dx := TargetX - X[i];
        InterpolatedY := A[i] + B[i]*dx + C[i]*dx*dx + D[i]*dx*dx*dx;
        RETURN;
    END_IF;
END_FOR;

// === Step 2: Handle edge case (TargetX >= X[10]) ===
IF TargetX >= X[10] THEN
    i := 9;
    dx := TargetX - X[i];
    InterpolatedY := A[i] + B[i]*dx + C[i]*dx*dx + D[i]*dx*dx*dx;
END_IF;
