FUNCTION_BLOCK SimplexSolver
VAR_INPUT
    C : ARRAY[1..3] OF REAL; // Objective function coefficients
    A : ARRAY[1..2, 1..3] OF REAL; // Constraint matrix
    B : ARRAY[1..2] OF REAL; // RHS values
    Execute : BOOL;
END_VAR

VAR_OUTPUT
    X : ARRAY[1..3] OF REAL; // Optimal solution vector
    ObjectiveValue : REAL; // Value of objective function
    Status : INT; // 0 = Not started, 1 = Success, 2 = Infeasible, 3 = Timeout/Error
END_VAR

VAR
    Tableau : ARRAY[0..2, 0..4] OF REAL; // Simplex tableau
    Iteration : INT;
    MaxIterations : INT := 20;
    PivotRow, PivotCol : INT;
    PivotElement : REAL;
    Temp : REAL;
    i, j : INT;
    Started : BOOL;
END_VAR

// Simplex procedure START
IF Execute AND NOT Started THEN
    // Step 1: Initialization of tableau
    // Row 0: Objective function
    FOR j := 1 TO 3 DO
        Tableau[0, j] := -C[j];
    END_FOR
    Tableau[0, 4] := 0.0;

    // Constraints
    FOR i := 1 TO 2 DO
        FOR j := 1 TO 3 DO
            Tableau[i, j] := A[i, j];
        END_FOR
        Tableau[i, 4] := B[i];
    END_FOR

    Iteration := 0;
    Status := 0;
    Started := TRUE;
END_IF

IF Started THEN
    // Step 2: Find pivot column (most negative in row 0)
    PivotCol := 1;
    FOR j := 2 TO 3 DO
        IF Tableau[0, j] < Tableau[0, PivotCol] THEN
            PivotCol := j;
        END_IF
    END_FOR

    IF Tableau[0, PivotCol] >= 0 THEN
        // Optimal solution found
        FOR j := 1 TO 3 DO
            X[j] := 0.0;
        END_FOR
        FOR i := 1 TO 2 DO
            FOR j := 1 TO 3 DO
                IF Tableau[i, j] = 1.0 THEN
                    X[j] := Tableau[i, 4];
                    EXIT;
                END_IF
            END_FOR
        END_FOR
        ObjectiveValue := Tableau[0, 4];
        Status := 1;
        Started := FALSE;
    ELSE
        // Step 3: Find pivot row (min ratio test)
        PivotRow := -1;
        FOR i := 1 TO 2 DO
            IF Tableau[i, PivotCol] > 0 THEN
                Temp := Tableau[i, 4] / Tableau[i, PivotCol];
                IF PivotRow = -1 OR Temp < (Tableau[PivotRow, 4] / Tableau[PivotRow, PivotCol]) THEN
                    PivotRow := i;
                END_IF
            END_IF
        END_FOR

        IF PivotRow = -1 THEN
            // Infeasible
            Status := 2;
            Started := FALSE;
        ELSE
            // Step 4: Pivot operation
            PivotElement := Tableau[PivotRow, PivotCol];
            FOR j := 1 TO 4 DO
                Tableau[PivotRow, j] := Tableau[PivotRow, j] / PivotElement;
            END_FOR
            FOR i := 0 TO 2 DO
                IF i <> PivotRow THEN
                    Temp := Tableau[i, PivotCol];
                    FOR j := 1 TO 4 DO
                        Tableau[i, j] := Tableau[i, j] - Temp * Tableau[PivotRow, j];
                    END_FOR
                END_IF
            END_FOR
        END_IF
    END_IF

    Iteration := Iteration + 1;
    IF Iteration > MaxIterations THEN
        Status := 3; // Timeout or too many iterations
        Started := FALSE;
    END_IF
END_IF
