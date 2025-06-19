(* Function Block: SplineInterpolator
   Purpose: Performs cubic spline interpolation for a set of data points.
   Features:
   - Inputs: X, Y (ARRAY[1..10] OF REAL, known points), NumPoints (UINT, number of points),
            TargetX (REAL, x-value to interpolate)
   - Outputs: InterpolatedY (REAL, interpolated y-value), Error (BOOL, TRUE if invalid)
   - Implements cubic spline with natural boundary conditions (c_0 = c_n = 0)
   - Precomputes coefficients (a_i, b_i, c_i, d_i) for each interval
   - Evaluates y = a_i + b_i*(x-x_i) + c_i*(x-x_i)^2 + d_i*(x-x_i)^3 at runtime
   - Optimized for PLCs: fixed-size arrays, minimal runtime computation, deterministic
   Mathematical Concept:
   - Cubic spline: Piecewise cubic polynomials ensuring C^2 continuity
   - For interval [x_i, x_i+1]: y(x) = a_i + b_i*(x-x_i) + c_i*(x-x_i)^2 + d_i*(x-x_i)^3
   - Natural spline: Second derivatives at endpoints are zero
   - Coefficients computed by solving tridiagonal system for c_i
   Notes:
   - Supports up to 10 points (9 intervals) to limit memory (~600 bytes)
   - Assumes 32-bit REAL arithmetic (IEEE 754 single-precision, ~7 digits)
   - Precomputation takes ~1-5ms; runtime evaluation <0.1ms
   - Suitable for robot trajectories, signal reconstruction, temperature profiles
   Challenges:
   - Memory: ~600 bytes for 10 points; scales with NumPoints
   - Precision: Limited by 32-bit REALs for large intervals or ill-conditioned points
   - Scan time: Precomputation may require slow task; runtime is fast
*)

FUNCTION_BLOCK SplineInterpolator
VAR_INPUT
    X : ARRAY[1..10] OF REAL;               (* Known x-coordinates, ascending *)
    Y : ARRAY[1..10] OF REAL;               (* Known y-values *)
    NumPoints : UINT;                       (* Number of points, 2..10 *)
    TargetX : REAL;                         (* Target x-value for interpolation *)
END_VAR
VAR_OUTPUT
    InterpolatedY : REAL;                   (* Interpolated y-value *)
    Error : BOOL;                           (* TRUE if input invalid or error *)
END_VAR
VAR
    (* Constants *)
    MaxPoints : UINT := 10;                 (* Maximum number of points *)
    Epsilon : REAL := 0.000001;             (* Threshold for numerical stability *)
    MaxReal : REAL := 1.0E6;                (* Overflow threshold *)
    
    (* Spline coefficients for each interval [x_i, x_i+1] *)
    A : ARRAY[1..9] OF REAL;                (* a_i = y_i *)
    B : ARRAY[1..9] OF REAL;                (* b_i coefficients *)
    C : ARRAY[1..9] OF REAL;                (* c_i coefficients *)
    D : ARRAY[1..9] OF REAL;                (* d_i coefficients *)
    
    (* Tridiagonal system variables *)
    H : ARRAY[1..9] OF REAL;                (* h_i = x_i+1 - x_i *)
    Alpha : ARRAY[1..9] OF REAL;            (* Right-hand side for c_i *)
    L : ARRAY[1..9] OF REAL;                (* Diagonal for tridiagonal *)
    Mu : ARRAY[1..9] OF REAL;               (* Off-diagonal for tridiagonal *)
    Z : ARRAY[1..9] OF REAL;                (* Temporary for c_i *)
    
    (* Working variables *)
    i, j : UINT;                            (* Loop indices *)
    DeltaX : REAL;                          (* x - x_i for polynomial *)
    Temp : REAL;                            (* Temporary calculation *)
    Valid : BOOL;                           (* Input validation flag *)
    Interval : UINT;                        (* Selected interval *)
END_VAR

(* Initialize outputs *)
InterpolatedY := 0.0;
Error := FALSE;
Valid := TRUE;

(* Validate inputs *)
IF NumPoints < 2 OR NumPoints > MaxPoints THEN
    Error := TRUE;
    Valid := FALSE;
ELSE
    (* Check if X is strictly increasing *)
    FOR i := 1 TO NumPoints - 1 DO
        IF X[i + 1] <= X[i] THEN
            Error := TRUE;
            Valid := FALSE;
            EXIT;
        END_IF
    END_FOR
END_IF

(* Precompute spline coefficients *)
IF Valid THEN
    (* Step 1: Compute h_i = x_i+1 - x_i *)
    FOR i := 1 TO NumPoints - 1 DO
        H[i] := X[i + 1] - X[i];
        IF H[i] < Epsilon THEN
            Error := TRUE;
            Valid := FALSE;
            EXIT;
        END_IF
    END_FOR
    
    (* Step 2: Set a_i = y_i *)
    FOR i := 1 TO NumPoints - 1 DO
        A[i] := Y[i];
    END_FOR
    
    (* Step 3: Compute alpha for tridiagonal system *)
    IF Valid THEN
        FOR i := 2 TO NumPoints - 1 DO
            Alpha[i] := 3.0 * ((Y[i + 1] - Y[i]) / H[i] - (Y[i] - Y[i - 1]) / H[i - 1]);
        END_FOR
        
        (* Step 4: Solve tridiagonal system for c_i *)
        L[1] := 1.0;  (* Natural spline: c_0 = 0 *)
        Mu[1] := 0.0;
        Z[1] := 0.0;
        
        FOR i := 2 TO NumPoints - 1 DO
            L[i] := 2.0 * (H[i - 1] + H[i]) - H[i - 1] * Mu[i - 1];
            IF ABS(L[i]) < Epsilon THEN
                Error := TRUE;
                Valid := FALSE;
                EXIT;
            END_IF
            Mu[i] := H[i] / L[i];
            Z[i] := (Alpha[i] - H[i - 1] * Z[i - 1]) / L[i];
        END_FOR
        
        IF Valid THEN
            L[NumPoints] := 1.0;  (* Natural spline: c_n = 0 *)
            Z[NumPoints] := 0.0;
            C[NumPoints - 1] := Z[NumPoints - 1];
            
            FOR i := NumPoints - 2 TO 1 BY -1 DO
                C[i] := Z[i] - Mu[i] * C[i + 1];
                IF ABS(C[i]) > MaxReal THEN
                    Error := TRUE;
                    Valid := FALSE;
                    EXIT;
                END_IF
            END_FOR
        END_IF
        
        (* Step 5: Compute b_i and d_i *)
        IF Valid THEN
            FOR i := 1 TO NumPoints - 1 DO
                B[i] := (Y[i + 1] - Y[i]) / H[i] - H[i] * (C[i + 1] + 2.0 * C[i]) / 3.0;
                D[i] := (C[i + 1] - C[i]) / (3.0 * H[i]);
                IF ABS(B[i]) > MaxReal OR ABS(D[i]) > MaxReal THEN
                    Error := TRUE;
                    Valid := FALSE;
                    EXIT;
                END_IF
            END_FOR
        END_IF
    END_IF
END_IF

(* Runtime evaluation *)
IF Valid THEN
    (* Find interval: X[i] <= TargetX < X[i+1] *)
    Interval := 0;
    IF TargetX < X[1] OR TargetX >= X[NumPoints] THEN
        (* Out of bounds *)
        Error := TRUE;
    ELSE
        FOR i := 1 TO NumPoints - 1 DO
            IF TargetX >= X[i] AND TargetX < X[i + 1] THEN
                Interval := i;
                EXIT;
            END_IF
        END_FOR
        IF Interval = 0 THEN
            (* Edge case: TargetX = X[NumPoints] *)
            InterpolatedY := Y[NumPoints];
        END_IF
    END_IF
    
    IF Interval > 0 THEN
        (* Evaluate spline: y = a_i + b_i*(x-x_i) + c_i*(x-x_i)^2 + d_i*(x-x_i)^3 *)
        DeltaX := TargetX - X[Interval];
        Temp := A[Interval] + DeltaX * (B[Interval] + DeltaX * (C[Interval] + DeltaX * D[Interval]));
        
        (* Check for numerical issues *)
        IF ABS(Temp) > MaxReal OR Temp <> Temp THEN
            Error := TRUE;
        ELSE
            InterpolatedY := Temp;
        END_IF
    END_IF
END_IF

(* Handle errors *)
IF Error THEN
    InterpolatedY := 0.0;
END_IF

END_FUNCTION_BLOCK
