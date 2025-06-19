(* Function Block: Cubic Spline Interpolation *)
(* Version: 1.0, Date: 2025-05-21 *)
(* Computes InterpolatedY for TargetX using cubic spline polynomial *)
FUNCTION_BLOCK FB_SplineInterpolator
VAR_INPUT
    Execute : BOOL;                   (* Trigger interpolation computation *)
    X : ARRAY[1..10] OF REAL;         (* Known x-values, must be sorted ascending *)
    Y : ARRAY[1..10] OF REAL;         (* Corresponding y-values *)
    A : ARRAY[1..9] OF REAL;          (* Spline coefficients A[i] for constant term *)
    B : ARRAY[1..9] OF REAL;          (* Spline coefficients B[i] for linear term *)
    C : ARRAY[1..9] OF REAL;          (* Spline coefficients C[i] for quadratic term *)
    D : ARRAY[1..9] OF REAL;          (* Spline coefficients D[i] for cubic term *)
    TargetX : REAL;                   (* X-value for interpolation *)
END_VAR

VAR_OUTPUT
    InterpolatedY : REAL;             (* Interpolated y-value *)
    Done : BOOL;                      (* Computation completed *)
    Error : BOOL;                     (* Error flag *)
    ErrorID : DWORD;                  (* Error code: 0x8001xxxx = Input error, 0x8002xxxx = Numerical *)
END_VAR

VAR
    N : UINT := 10;                   (* Number of data points *)
    i : UINT;                         (* Loop index *)
    Interval : UINT;                  (* Index of interval [X[i], X[i+1]] *)
    DeltaX : REAL;                    (* TargetX - X[i] for polynomial evaluation *)
    Found : BOOL;                     (* Flag for interval search *)
    Temp : REAL;                      (* Temporary variable for polynomial terms *)
END_VAR

(* Initialize outputs *)
InterpolatedY := 0.0;
Done := FALSE;
Error := FALSE;
ErrorID := 0;

(* Main logic *)
IF Execute THEN
    (* Validate inputs *)
    (* Ensure X is sorted and within reasonable range *)
    FOR i := 1 TO N - 1 DO
        IF X[i] >= X[i + 1] THEN
            Error := TRUE;
            ErrorID := 16#80010001;   (* X not sorted ascending *)
            Done := TRUE;
            RETURN;
        END_IF;
        IF ABS(X[i]) > 1.0E10 OR ABS(Y[i]) > 1.0E10 OR 
           ABS(A[i]) > 1.0E10 OR ABS(B[i]) > 1.0E10 OR 
           ABS(C[i]) > 1.0E10 OR ABS(D[i]) > 1.0E10 THEN
            Error := TRUE;
            ErrorID := 16#80010002;   (* Input values too large *)
            Done := TRUE;
            RETURN;
        END_IF;
    END_FOR;
    IF ABS(X[N]) > 1.0E10 OR ABS(Y[N]) > 1.0E10 OR ABS(TargetX) > 1.0E10 THEN
        Error := TRUE;
        ErrorID := 16#80010002;       (* Input values too large *)
        Done := TRUE;
        RETURN;
    END_IF;

    (* Find interval: X[i] <= TargetX < X[i+1] *)
    Interval := 0;
    Found := FALSE;
    FOR i := 1 TO N - 1 DO
        IF TargetX >= X[i] AND TargetX < X[i + 1] THEN
            Interval := i;
            Found := TRUE;
            EXIT;
        END_IF;
    END_FOR;

    (* Handle boundary conditions and extrapolation *)
    IF NOT Found THEN
        IF TargetX < X[1] OR TargetX >= X[N] THEN
            Error := TRUE;
            ErrorID := 16#80010003;   (* TargetX outside range, extrapolation not supported *)
            InterpolatedY := 0.0;     (* Fallback to zero *)
            Done := TRUE;
            RETURN;
        END_IF;
    END_IF;

    (* Compute interpolated value using cubic spline polynomial *)
    (* Formula: Y = A[i] + B[i]*(x - X[i]) + C[i]*(x - X[i])^2 + D[i]*(x - X[i])^3 *)
    DeltaX := TargetX - X[Interval];  (* Compute x - X[i] *)
    
    (* Evaluate polynomial *)
    Temp := DeltaX * DeltaX;          (* (x - X[i])^2 *)
    InterpolatedY := A[Interval] +     (* Constant term *)
                    B[Interval] * DeltaX + (* Linear term *)
                    C[Interval] * Temp +   (* Quadratic term *)
                    D[Interval] * Temp * DeltaX; (* Cubic term *)

    (* Check result for numerical validity *)
    IF ABS(InterpolatedY) > 1.0E10 THEN
        Error := TRUE;
        ErrorID := 16#80020001;       (* Numerical overflow in result *)
        InterpolatedY := 0.0;         (* Fallback to zero *)
    END_IF;

    Done := TRUE;                     (* Mark computation as complete *)
ELSE
    (* Reset outputs when Execute is FALSE *)
    InterpolatedY := 0.0;
    Done := FALSE;
    Error := FALSE;
    ErrorID := 0;
END_IF;

END_FUNCTION_BLOCK
