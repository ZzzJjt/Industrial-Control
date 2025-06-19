FUNCTION_BLOCK ComputeEigenvalue_10x10
VAR_INPUT
    A : ARRAY[1..10, 1..10] OF REAL; (* 10x10 input matrix *)
    Start : BOOL;                   (* Trigger to start computation *)
END_VAR
VAR_OUTPUT
    Eigenvalue : REAL;             (* Approximated dominant eigenvalue *)
    Done : BOOL;                   (* TRUE when computation finishes *)
END_VAR
VAR
    x : ARRAY[1..10] OF REAL := [1,1,1,1,1,1,1,1,1,1];  (* Initial vector *)
    y : ARRAY[1..10] OF REAL;
    norm : REAL := 1.0;
    iter : INT := 0;
    MaxIter : INT := 20;           (* Max iterations for convergence *)
    tol : REAL := 0.0001;          (* Convergence tolerance *)
    diff : REAL := 1.0;
    lambda_old : REAL := 0.0;
    lambda_new : REAL := 0.0;
    i, j : INT;
    running : BOOL := FALSE;
END_VAR

(* === Core Power Method Algorithm === *)
IF Start AND NOT running THEN
    running := TRUE;
    iter := 0;
    diff := 1.0;
END_IF

IF running THEN
    (* Step 1: Multiply A * x => y *)
    FOR i := 1 TO 10 DO
        y[i] := 0.0;
        FOR j := 1 TO 10 DO
            y[i] := y[i] + A[i,j] * x[j];
        END_FOR
    END_FOR

    (* Step 2: Normalize y *)
    norm := 0.0;
    FOR i := 1 TO 10 DO
        norm := norm + y[i]*y[i];
    END_FOR
    norm := SQRT(norm);
    IF norm > 0.000001 THEN
        FOR i := 1 TO 10 DO
            y[i] := y[i] / norm;
        END_FOR
    END_IF

    (* Step 3: Estimate eigenvalue λ = (x' * A * x) *)
    lambda_new := 0.0;
    FOR i := 1 TO 10 DO
        lambda_new := lambda_new + x[i] * y[i] * norm;  (* Rayleigh quotient *)
    END_FOR

    (* Step 4: Check convergence *)
    diff := ABS(lambda_new - lambda_old);
    lambda_old := lambda_new;

    (* Step 5: Update x := y *)
    FOR i := 1 TO 10 DO
        x[i] := y[i];
    END_FOR

    iter := iter + 1;
    IF (diff < tol) OR (iter >= MaxIter) THEN
        Eigenvalue := lambda_new;
        Done := TRUE;
        running := FALSE;
    END_IF
END_IF
