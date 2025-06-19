FUNCTION_BLOCK SimplexSolver
VAR_INPUT
    NumVars      : INT; // Number of decision variables (<=3)
    NumConstraints : INT; // Number of constraints (<=3)
    C            : ARRAY[1..3] OF REAL; // Objective function coefficients
    A            : ARRAY[1..3, 1..3] OF REAL; // Constraint coefficients
    B            : ARRAY[1..3] OF REAL; // RHS values
END_VAR
VAR_OUTPUT
    X            : ARRAY[1..3] OF REAL; // Optimal solution
    OptValue     : REAL; // Objective value
    Status       : INT; // 0: optimal, 1: infeasible, 2: iteration limit reached
END_VAR
VAR
    Tableau      : ARRAY[0..3, 0..6] OF REAL; // Simplex tableau
    Row, Col, i, j : INT;
    PivotCol, PivotRow : INT;
    MinRatio     : REAL;
    IterCount    : INT := 0;
    MAX_ITER     : INT := 20;
    Temp         : REAL;
END_VAR

// Initialization
// Objective function row (row 0)
FOR j := 1 TO NumVars DO
    Tableau[0, j] := -C[j];
END_FOR

// Constraints and slack variables
FOR i := 1 TO NumConstraints DO
    FOR j := 1 TO NumVars DO
        Tableau[i, j] := A[i, j];
    END_FOR
    Tableau[i, NumVars + i] := 1.0; // Slack variable
    Tableau[i, 0] := B[i]; // RHS
END_FOR

// Simplex iterations
WHILE (IterCount < MAX_ITER) DO
    // Step 1: Find entering variable (most negative coefficient)
    PivotCol := 1;
    FOR j := 2 TO NumVars + NumConstraints DO
        IF Tableau[0, j] < Tableau[0, PivotCol] THEN
            PivotCol := j;
        END_IF
    END_FOR
    IF Tableau[0, PivotCol] >= 0 THEN
        Status := 0; // Optimal
        EXIT;
    END_IF

    // Step 2: Find leaving variable (minimum positive ratio)
    MinRatio := 1.0E+10;
    PivotRow := -1;
    FOR i := 1 TO NumConstraints DO
        IF Tableau[i, PivotCol] > 0 THEN
            Temp := Tableau[i, 0] / Tableau[i, PivotCol];
            IF Temp < MinRatio THEN
                MinRatio := Temp;
                PivotRow := i;
            END_IF
        END_IF
    END_FOR

    IF PivotRow = -1 THEN
        Status := 1; // Infeasible
        EXIT;
    END_IF

    // Step 3: Pivot operation
    Temp := Tableau[PivotRow, PivotCol];
    FOR j := 0 TO NumVars + NumConstraints DO
        Tableau[PivotRow, j] := Tableau[PivotRow, j] / Temp;
    END_FOR

    FOR i := 0 TO NumConstraints DO
        IF i <> PivotRow THEN
            Temp := Tableau[i, PivotCol];
            FOR j := 0 TO NumVars + NumConstraints DO
                Tableau[i, j] := Tableau[i, j] - Temp * Tableau[PivotRow, j];
            END_FOR
        END_IF
    END_FOR

    IterCount := IterCount + 1;
END_WHILE

IF IterCount >= MAX_ITER THEN
    Status := 2; // Iteration limit reached
END_IF

// Extract solution
FOR j := 1 TO NumVars DO
    X[j] := 0.0;
    FOR i := 1 TO NumConstraints DO
        IF Tableau[i, j] = 1.0 THEN
            X[j] := Tableau[i, 0];
            EXIT;
        END_IF
    END_FOR
END_FOR

OptValue := Tableau[0, 0];
