FUNCTION_BLOCK FB_EigenvalueSolver_10x10
VAR_INPUT
    MatrixInput : ARRAY[1..10, 1..10] OF REAL; // Input 10x10 matrix
    MaxIterations : DINT := 50;                // Max iterations per eigenvalue
    Tolerance : REAL := 0.0001;                 // Convergence threshold
    Enable : BOOL := FALSE;                     // Enable computation
END_VAR

VAR_OUTPUT
    EigenValues : ARRAY[1..10] OF REAL := [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0];
    ConvergedCount : DINT := 0;                  // Number of eigenvalues found
    ExecutionTimeMS : REAL := 0.0;              // Time taken in milliseconds
    ErrorFlag : BOOL := FALSE;                  // TRUE if error occurred
    NotConverged : BOOL := FALSE;               // TRUE if some didn't converge
END_VAR

VAR
    // Internal variables
    i, j, k, m, iter: DINT;
    norm, diff, lambda, startTime: REAL;

    A : ARRAY[1..10, 1..10] OF REAL := [[0.0]];     // Working copy of matrix
    v : ARRAY[1..10] OF REAL := [0.0];              // Eigenvector estimate
    v_new : ARRAY[1..10] OF REAL := [0.0];          // Updated eigenvector
    tempRow : ARRAY[1..10] OF REAL := [0.0];        // Row storage during deflation
    diagSum : REAL := 0.0;
    converged: BOOL := FALSE;
    timerStart: TIME := T#0ms;
END_VAR

// Main logic
IF Enable THEN
    startTime := TIME_TO_REAL(TIME());
    
    // Copy input matrix into working memory
    FOR i := 1 TO 10 DO
        FOR j := 1 TO 10 DO
            A[i, j] := MatrixInput[i, j];
        END_FOR;
    END_FOR;

    diagSum := 0.0;
    FOR m := 1 TO 10 DO
        // Initialize starting vector as all ones
        FOR i := 1 TO 10 DO
            v[i] := 1.0;
        END_FOR;

        // Normalize initial vector
        norm := 0.0;
        FOR i := 1 TO 10 DO
            norm := norm + v[i] * v[i];
        END_FOR;
        IF norm > 0.0 THEN
            FOR i := 1 TO 10 DO
                v[i] := v[i] / SQRT(norm);
            END_FOR;
        END_IF;

        // Power iteration loop
        iter := 0;
        converged := FALSE;
        WHILE NOT converged AND (iter < MaxIterations) DO
            // Multiply A * v -> v_new
            FOR i := 1 TO 10 DO
                v_new[i] := 0.0;
                FOR j := 1 TO 10 DO
                    v_new[i] := v_new[i] + A[i, j] * v[j];
                END_FOR;
            END_FOR;

            // Normalize new vector
            norm := 0.0;
            FOR i := 1 TO 10 DO
                norm := norm + v_new[i] * v_new[i];
            END_FOR;
            IF norm > 0.0 THEN
                FOR i := 1 TO 10 DO
                    v[i] := v_new[i] / SQRT(norm);
                END_FOR;
            END_IF;

            // Estimate eigenvalue using Rayleigh quotient
            lambda := 0.0;
            FOR i := 1 TO 10 DO
                lambda := lambda + v[i] * v_new[i];
            END_FOR;

            // Check convergence: difference between old and new eigenvalue
            diff := ABS(lambda - EigenValues[m]);
            IF diff < Tolerance THEN
                converged := TRUE;
            END_IF;

            EigenValues[m] := lambda;
            iter := iter + 1;
        END_WHILE;

        IF NOT converged THEN
            NotConverged := TRUE;
        ELSE
            ConvergedCount := m;
        END_IF;

        // Deflate matrix to find next eigenvalue
        FOR i := 1 TO 10 DO
            FOR j := 1 TO 10 DO
                A[i, j] := A[i, j] - lambda * v[i] * v[j];
            END_FOR;
        END_FOR;
    END_FOR;

    // Calculate execution time
    ExecutionTimeMS := (TIME_TO_REAL(TIME()) - startTime) * 1000.0;

ELSE
    // Reset outputs when disabled
    FOR i := 1 TO 10 DO
        EigenValues[i] := 0.0;
    END_FOR;
    ConvergedCount := 0;
    NotConverged := FALSE;
    ExecutionTimeMS := 0.0;
END_IF;

// Set error flag if not all converged
ErrorFlag := NotConverged;

PROGRAM PLC_PRG
VAR
    EigenSolver: FB_EigenvalueSolver_10x10;

    TestMatrix : ARRAY[1..10, 1..10] OF REAL :=
        [[2.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0],
         [0.0, 3.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0],
         [0.0, 0.0, 4.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0],
         [0.0, 0.0, 0.0, 5.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0],
         [0.0, 0.0, 0.0, 0.0, 6.0, 0.0, 0.0, 0.0, 0.0, 0.0],
         [0.0, 0.0, 0.0, 0.0, 0.0, 7.0, 0.0, 0.0, 0.0, 0.0],
         [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 8.0, 0.0, 0.0, 0.0],
         [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 9.0, 0.0, 0.0],
         [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 10.0, 0.0],
         [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 11.0]];

    ResultsReady: BOOL := FALSE;
END_VAR

// Run solver
EigenSolver(
    MatrixInput := TestMatrix,
    MaxIterations := 100,
    Tolerance := 0.0001,
    Enable := TRUE
);

ResultsReady := EigenSolver.ConvergedCount = 10;
