FUNCTION_BLOCK SplineInterpolator
VAR_INPUT
    X : ARRAY[1..10] OF REAL; (* Known x-values, must be strictly increasing *)
    Y : ARRAY[1..10] OF REAL; (* Known y-values *)
    A : ARRAY[1..9] OF REAL; (* Spline coefficients: constant term (y[i]) *)
    B : ARRAY[1..9] OF REAL; (* Spline coefficients: linear term *)
    C : ARRAY[1..9] OF REAL; (* Spline coefficients: quadratic term *)
    D : ARRAY[1..9] OF REAL; (* Spline coefficients: cubic term *)
    TargetX : REAL; (* x-position for interpolation *)
END_VAR
VAR_OUTPUT
    InterpolatedY : REAL; (* Interpolated y-value at TargetX *)
    ErrorCode : INT := 0; (* 0: Success, 1: Invalid input, 2: Out of range *)
END_VAR
VAR
    i, j : INT; (* Loop indices *)
    N : INT := 10; (* Number of data points *)
    Interval : INT; (* Index of interval [X[i], X[i+1]] *)
    DeltaX : REAL; (* TargetX - X[i] for polynomial evaluation *)
    Found : BOOL; (* Flag for interval search *)
END_VAR

(* Initialize outputs *)
InterpolatedY := 0.0;
ErrorCode := 0;

(* Validate inputs *)
FOR i := 1 TO N DO
    IF NOT IS_VALID_REAL(X[i]) OR NOT IS_VALID_REAL(Y[i]) THEN
        ErrorCode := 1; (* Invalid x or y values *)
        RETURN;
    END_IF;
END_FOR;
FOR i := 1 TO N-1 DO
    IF NOT IS_VALID_REAL(A[i]) OR NOT IS_VALID_REAL(B[i]) OR 
       NOT IS_VALID_REAL(C[i]) OR NOT IS_VALID_REAL(D[i]) THEN
        ErrorCode := 1; (* Invalid coefficients *)
        RETURN;
    END_IF;
    IF X[i] >= X[i+1] THEN
        ErrorCode := 1; (* X must be strictly increasing *)
        RETURN;
    END_IF;
END_FOR;
IF NOT IS_VALID_REAL(TargetX) THEN
    ErrorCode := 1; (* Invalid TargetX *)
    RETURN;
END_IF;

(* Check if TargetX is within range [X[1], X[N]] *)
IF TargetX < X[1] OR TargetX > X[N] THEN
    ErrorCode := 2; (* Out of range *)
    RETURN;
END_IF;

(* Find interval: X[i] <= TargetX < X[i+1] *)
Interval := 0;
Found := FALSE;
FOR i := 1 TO N-1 DO
    IF TargetX >= X[i] AND TargetX < X[i+1] THEN
        Interval := i;
        Found := TRUE;
        EXIT;
    END_IF;
END_FOR;
IF NOT Found THEN
    (* Handle edge case: TargetX = X[N] *)
    IF ABS(TargetX - X[N]) < 1.0E-6 THEN
        InterpolatedY := Y[N];
        RETURN;
    ELSE
        ErrorCode := 2; (* Should not occur due to range check *)
        RETURN;
    END_IF;
END_IF;

(* Evaluate cubic polynomial: y = a + b*(x-x_i) + c*(x-x_i)^2 + d*(x-x_i)^3 *)
DeltaX := TargetX - X[Interval];
InterpolatedY := A[Interval] + 
                B[Interval] * DeltaX + 
                C[Interval] * DeltaX * DeltaX + 
                D[Interval] * DeltaX * DeltaX * DeltaX;

(* Validate result *)
IF NOT IS_VALID_REAL(InterpolatedY) THEN
    ErrorCode := 1; (* Invalid result *)
    InterpolatedY := 0.0;
END_IF;

(* Helper function to check if a REAL value is valid *)
METHOD IS_VALID_REAL : BOOL
VAR_INPUT
    Value : REAL;
END_VAR
IS_VALID_REAL := NOT (Value = 0.0 / 0.0) AND NOT (Value = 1.0 / 0.0);
END_METHOD

END_FUNCTION_BLOCK
