FUNCTION_BLOCK SimplexSolver
VAR_INPUT
    C : ARRAY[1..3] OF REAL;            // Objective coefficients
    A : ARRAY[1..3, 1..3] OF REAL;      // Constraint coefficients
    B : ARRAY[1..3] OF REAL;            // Right-hand side values
    MaxIterations : INT := 20;         // Iteration safety limit
END_VAR

VAR_OUTPUT
    X : ARRAY[1..3] OF REAL;           // Solution vector
    OptimalValue : REAL;              // Value of objective function
    Status : INT;                     // 0 = running, 1 = optimal, -1 = infeasible
END_VAR

VAR
    Tableau : ARRAY[0..3, 0..4] OF REAL;  // Simplex tableau [row][col]
    Iter : INT;
    PivotCol : INT;
    PivotRow : INT;
    Ratio : REAL;
    MinRatio : REAL;
    i, j : INT;
END_VAR

// Step 0: Initialize the simplex tableau
IF Status = 0 THEN
    // Fill tableau with A and B
    FOR i := 1 TO 3 DO
        FOR j := 1 TO 3 DO
            Tableau[i, j] := A[i, j];
        END_FOR
        Tableau[i, 4] := B[i]; // RHS
    END_FOR

    // Fill objective row (negated for maximization)
    FOR j := 1 TO 3 DO
        Tableau[0, j] := -C[j];
    END_FOR
    Tableau[0, 4] := 0.0;

    // Start iterations
    FOR Iter := 1 TO MaxIterations DO

        // Step 1: Find pivot column (most negative)
        PivotCol := 1;
        FOR j := 2 TO 3 DO
            IF Tableau[0, j] < Tableau[0, PivotCol] THEN
                PivotCol := j;
            END_IF
        END_FOR

        // Step 2: If no negative values, solution is optimal
        IF Tableau[0, PivotCol] >= 0 THEN
            Status := 1;
            EXIT;
        END_IF

        // Step 3: Find pivot row (min ratio test)
        MinRatio := 1.0E+6;
        PivotRow := 0;
        FOR i := 1 TO 3 DO
            IF Tableau[i, PivotCol] > 0 THEN
                Ratio := Tableau[i, 4] / Tableau[i, PivotCol];
                IF Ratio < MinRatio THEN
                    MinRatio := Ratio;
                    PivotRow := i;
                END_IF
            END_IF
        END_FOR

        // Step 4: Check for infeasibility
        IF PivotRow = 0 THEN
            Status := -1;
            EXIT;
        END_IF

        // Step 5: Normalize pivot row
        Ratio := Tableau[PivotRow, PivotCol];
        FOR j := 1 TO 4 DO
            Tableau[PivotRow, j] := Tableau[PivotRow, j] / Ratio;
        END_FOR

        // Step 6: Eliminate other rows
        FOR i := 0 TO 3 DO
            IF i <> PivotRow THEN
                Ratio := Tableau[i, PivotCol];
                FOR j := 1 TO 4 DO
                    Tableau[i, j] := Tableau[i, j] - Ratio * Tableau[PivotRow, j];
                END_FOR
            END_IF
        END_FOR

    END_FOR // iteration loop

    // Step 7: Read solution
    IF Status = 1 THEN
        FOR j := 1 TO 3 DO
            X[j] := 0.0;
            FOR i := 1 TO 3 DO
                IF Tableau[i, j] = 1.0 THEN
                    X[j] := Tableau[i, 4];
                    EXIT;
                END_IF
            END_FOR
        END_FOR
        OptimalValue := Tableau[0, 4];
    END_IF
END_IF
