FUNCTION_BLOCK SimplexSolver
VAR_INPUT
    Execute : BOOL; // Rising edge starts the simplex solver
    C : ARRAY[1..3] OF REAL; // Coefficients of the objective function
    A : ARRAY[1..3, 1..3] OF REAL; // Constraint matrix
    B : ARRAY[1..3] OF REAL; // Right-hand side
END_VAR

VAR_OUTPUT
    X : ARRAY[1..3] OF REAL; // Optimal values for variables
    ObjValue : REAL; // Final value of objective function
    Status : INT; // 0 = Idle, 1 = Running, 2 = Optimal, 3 = Infeasible, 4 = MaxIterations
END_VAR

VAR
    Tableau : ARRAY[0..3, 0..4] OF REAL; // Simplex tableau (3 constraints + 1 row, 3 vars + 1 RHS)
    Row, Col, i, j, k : INT;
    PivotRow, PivotCol : INT;
    MinVal, Factor : REAL;
    Iteration : INT := 0;
    MaxIter : INT := 20;
    Triggered : BOOL := FALSE;
END_VAR

// ------------------- Main Logic -------------------
IF Execute AND NOT Triggered THEN
    Triggered := TRUE;
    Status := 1; // Running
    Iteration := 0;

    // Initialize tableau
    FOR i := 1 TO 3 DO
        FOR j := 1 TO 3 DO
            Tableau[i, j] := A[i, j];
        END_FOR
        Tableau[i, 4] := B[i]; // RHS
    END_FOR

    // Objective function row (row 0)
    FOR j := 1 TO 3 DO
        Tableau[0, j] := -C[j]; // Negate for maximization
    END_FOR
    Tableau[0, 4] := 0.0; // RHS of objective

END_IF

IF Triggered AND Status = 1 THEN

    // ---- Find pivot column (most negative coefficient in row 0) ----
    MinVal := 0.0;
    PivotCol := 0;
    FOR j := 1 TO 3 DO
        IF Tableau[0, j] < MinVal THEN
            MinVal := Tableau[0, j];
            PivotCol := j;
        END_IF
    END_FOR

    IF PivotCol = 0 THEN
        // No negative coefficients: optimal solution found
        FOR j := 1 TO 3 DO
            X[j] := 0.0;
            FOR i := 1 TO 3 DO
                IF Tableau[i, j] = 1.0 THEN
                    X[j] := Tableau[i, 4]; // Basic variable
                    EXIT;
                END_IF
            END_FOR
        END_FOR
        ObjValue := Tableau[0, 4];
        Status := 2; // Optimal
        Triggered := FALSE;
        RETURN;
    END_IF

    // ---- Find pivot row (min ratio of RHS / pivot column) ----
    PivotRow := 0;
    MinVal := 1.0E+10;
    FOR i := 1 TO 3 DO
        IF Tableau[i, PivotCol] > 0.0 THEN
            Factor := Tableau[i, 4] / Tableau[i, PivotCol];
            IF Factor < MinVal THEN
                MinVal := Factor;
                PivotRow := i;
            END_IF
        END_IF
    END_FOR

    IF PivotRow = 0 THEN
        Status := 3; // Infeasible
        Triggered := FALSE;
        RETURN;
    END_IF

    // ---- Pivot operation ----
    Factor := Tableau[PivotRow, PivotCol];
    FOR j := 1 TO 4 DO
        Tableau[PivotRow, j] := Tableau[PivotRow, j] / Factor; // Normalize pivot row
    END_FOR

    FOR i := 0 TO 3 DO
        IF i <> PivotRow THEN
            Factor := Tableau[i, PivotCol];
            FOR j := 1 TO 4 DO
                Tableau[i, j] := Tableau[i, j] - Factor * Tableau[PivotRow, j];
            END_FOR
        END_IF
    END_FOR

    Iteration := Iteration + 1;
    IF Iteration >= MaxIter THEN
        Status := 4; // Max Iterations reached
        Triggered := FALSE;
    END_IF

END_IF
