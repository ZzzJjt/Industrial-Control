FUNCTION_BLOCK SimplexSolver
VAR_INPUT
    Execute        : BOOL; // Rising-edge trigger to start solving
    C              : ARRAY[1..5] OF REAL; // Objective coefficients
    A              : ARRAY[1..5, 1..5] OF REAL; // Constraint coefficients
    B              : ARRAY[1..5] OF REAL; // RHS values
    NumVars        : INT;  // Number of variables (N ≤ 5)
    NumConstraints : INT;  // Number of constraints (M ≤ 5)
    MaxIterations  : INT := 50; // Loop safety
END_VAR

VAR_OUTPUT
    X              : ARRAY[1..5] OF REAL; // Solution vector
    ObjectiveValue : REAL; // Optimal value
    Done           : BOOL;
    Error          : BOOL;
    StatusCode     : INT; // 0=OK, 1=Infeasible, 2=MaxIter
END_VAR

VAR
    Tableau        : ARRAY[0..6, 0..6] OF REAL; // M+1 x N+1
    Basis          : ARRAY[1..5] OF INT;
    Iter, i, j, pivotRow, pivotCol : INT;
    Pivot, Ratio, MinRatio : REAL;
    ExecLatch      : BOOL;
END_VAR

// --- Step 1: Edge Detection ---
IF Execute AND NOT ExecLatch THEN
    ExecLatch := TRUE;
    Done := FALSE;
    Error := FALSE;
    StatusCode := 0;

    // --- Step 2: Initialize Simplex Tableau ---
    FOR i := 1 TO NumConstraints DO
        FOR j := 1 TO NumVars DO
            Tableau[i, j] := A[i, j];
        END_FOR
        Tableau[i, NumVars+1] := B[i]; // RHS
    END_FOR
    FOR j := 1 TO NumVars DO
        Tableau[0, j] := -C[j]; // Objective row
    END_FOR
    // RHS of objective row is 0 by default
    FOR i := 1 TO NumConstraints DO
        Basis[i] := NumVars + i;
    END_FOR

    // --- Step 3: Main Iteration ---
    FOR Iter := 1 TO MaxIterations DO
        // --- Find pivot column (most negative coefficient) ---
        pivotCol := 0;
        FOR j := 1 TO NumVars DO
            IF Tableau[0, j] < 0.0 THEN
                pivotCol := j;
                EXIT;
            END_IF
        END_FOR

        // All entries ≥ 0: Optimal reached
        IF pivotCol = 0 THEN
            EXIT;
        END_IF

        // --- Find pivot row (min ratio test) ---
        MinRatio := 1.0E10;
        pivotRow := 0;
        FOR i := 1 TO NumConstraints DO
            IF Tableau[i, pivotCol] > 0.0 THEN
                Ratio := Tableau[i, NumVars+1] / Tableau[i, pivotCol];
                IF Ratio < MinRatio THEN
                    MinRatio := Ratio;
                    pivotRow := i;
                END_IF
            END_IF
        END_FOR

        // No valid pivot row: Infeasible
        IF pivotRow = 0 THEN
            StatusCode := 1;
            Error := TRUE;
            EXIT;
        END_IF

        // --- Pivot operation ---
        Pivot := Tableau[pivotRow, pivotCol];
        FOR j := 1 TO NumVars+1 DO
            Tableau[pivotRow, j] := Tableau[pivotRow, j] / Pivot;
        END_FOR

        FOR i := 0 TO NumConstraints DO
            IF i <> pivotRow THEN
                Ratio := Tableau[i, pivotCol];
                FOR j := 1 TO NumVars+1 DO
                    Tableau[i, j] := Tableau[i, j] - Ratio * Tableau[pivotRow, j];
                END_FOR
            END_IF
        END_FOR

        Basis[pivotRow] := pivotCol;
    END_FOR

    // --- Step 4: Output Result ---
    IF Iter > MaxIterations THEN
        StatusCode := 2;
        Error := TRUE;
    ELSE
        FOR i := 1 TO NumVars DO
            X[i] := 0.0;
        END_FOR
        FOR i := 1 TO NumConstraints DO
            IF Basis[i] <= NumVars THEN
                X[Basis[i]] := Tableau[i, NumVars+1];
            END_IF
        END_FOR
        ObjectiveValue := Tableau[0, NumVars+1];
        Done := TRUE;
    END_IF
END_IF

IF NOT Execute THEN
    ExecLatch := FALSE;
END_IF
