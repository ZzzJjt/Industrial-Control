FUNCTION_BLOCK ComputeEigenvalue_10x10
VAR_INPUT
    A : ARRAY[1..10, 1..10] OF REAL; // Input matrix
    MaxIterations : INT := 50;       // Max iteration limit
    Epsilon : REAL := 0.0001;        // Convergence threshold
END_VAR
VAR_OUTPUT
    EigenValue : REAL;               // Estimated dominant eigenvalue
    Converged  : BOOL;               // TRUE if convergence achieved
END_VAR
VAR
    x      : ARRAY[1..10] OF REAL;   // Current vector
    x_next : ARRAY[1..10] OF REAL;   // Next iteration vector
    lambda_old : REAL := 0.0;
    lambda_new : REAL := 0.0;
    dot_xx : REAL;
    dot_Ax : REAL;
    norm   : REAL;
    i, j, iter : INT;
END_VAR

// Initialization
IF NOT Converged THEN
    // Start with initial vector [1,1,...,1]
    FOR i := 1 TO 10 DO
        x[i] := 1.0;
    END_FOR

    // Begin iteration loop
    FOR iter := 1 TO MaxIterations DO
        // Multiply A*x
        FOR i := 1 TO 10 DO
            x_next[i] := 0.0;
            FOR j := 1 TO 10 DO
                x_next[i] := x_next[i] + A[i, j] * x[j];
            END_FOR
        END_FOR

        // Normalize x_next
        norm := 0.0;
        FOR i := 1 TO 10 DO
            norm := norm + x_next[i] * x_next[i];
        END_FOR
        norm := SQRT(norm);
        FOR i := 1 TO 10 DO
            x_next[i] := x_next[i] / norm;
        END_FOR

        // Estimate eigenvalue: λ = (x' * A * x)
        lambda_new := 0.0;
        FOR i := 1 TO 10 DO
            dot_xx := 0.0;
            FOR j := 1 TO 10 DO
                dot_xx := dot_xx + A[i,j] * x_next[j];
            END_FOR
            lambda_new := lambda_new + x_next[i] * dot_xx;
        END_FOR

        // Check for convergence
        IF ABS(lambda_new - lambda_old) < Epsilon THEN
            Converged := TRUE;
            EXIT;
        END_IF

        // Prepare for next iteration
        FOR i := 1 TO 10 DO
            x[i] := x_next[i];
        END_FOR
        lambda_old := lambda_new;
    END_FOR

    // Set output
    EigenValue := lambda_new;
END_IF
