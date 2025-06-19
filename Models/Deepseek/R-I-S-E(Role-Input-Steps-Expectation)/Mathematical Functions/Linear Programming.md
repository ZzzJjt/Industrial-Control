FUNCTION_BLOCK FB_SimplexSolver
VAR_INPUT
    // Objective coefficients (maximization assumed)
    C : ARRAY[1..5] OF REAL := [0.0, 0.0, 0.0, 0.0, 0.0];

    // Constraint matrix (M rows x N columns)
    A : ARRAY[1..5, 1..5] OF REAL :=
        [[0.0, 0.0, 0.0, 0.0, 0.0],
         [0.0, 0.0, 0.0, 0.0, 0.0],
         [0.0, 0.0, 0.0, 0.0, 0.0],
         [0.0, 0.0, 0.0, 0.0, 0.0],
         [0.0, 0.0, 0.0, 0.0, 0.0]];

    B : ARRAY[1..5] OF REAL := [0.0, 0.0, 0.0, 0.0, 0.0]; // RHS values

    NumVars : DINT := 2;       // Number of decision variables (N)
    NumCons : DINT := 2;       // Number of constraints (M)
    Enable : BOOL := FALSE;    // Trigger solver
END_VAR

VAR_OUTPUT
    X : ARRAY[1..5] OF REAL := [0.0, 0.0, 0.0, 0.0, 0.0]; // Solution vector
    OptimalValue : REAL := 0.0;
    Status : INT := 0; // 0 = solution found, 1 = infeasible, 2 = unbounded, 3 = max iterations
END_VAR

VAR
    // Tableau storage (rows = M + 1, cols = M + N + 1)
    tableau : ARRAY[0..6, 0..10] OF REAL := [[0.0]];

    i, j, k, m, n, row, col : DINT;
    pivotVal, ratio, minRatio : REAL;
    pivotCol, pivotRow : DINT;
    hasNegative : BOOL := FALSE;
    done : BOOL := FALSE;
    iterCount, maxIter : DINT := 50;
    epsilon : REAL := 1.0E-6;
END_VAR

// Main logic
IF Enable THEN

    // Reset outputs
    FOR i := 1 TO 5 DO
        X[i] := 0.0;
    END_FOR;
    OptimalValue := 0.0;
    Status := 0;

    // Initialize tableau dimensions
    m := NumCons;
    n := NumVars;

    // Step 1: Fill tableau with objective function and slack variables
    FOR row := 0 TO m DO
        FOR col := 0 TO m + n DO
            tableau[row, col] := 0.0;
        END_FOR;
    END_FOR;

    // Objective row (negated for maximization)
    FOR j := 0 TO n - 1 DO
        tableau[0, j] := -C[j + 1];
    END_FOR;

    // Constraints
    FOR i := 1 TO m DO
        FOR j := 0 TO n - 1 DO
            tableau[i, j] := A[i, j + 1];
        END_FOR;
        tableau[i, n + i] := 1.0; // Slack variable
        tableau[i, m + n] := B[i];
    END_FOR;

    // Simplex loop
    iterCount := 0;
    WHILE NOT done AND (iterCount < maxIter) DO
        iterCount := iterCount + 1;

        // Step 2: Find entering variable (most negative in objective row)
        pivotCol := -1;
        FOR j := 0 TO m + n - 1 DO
            IF tableau[0, j] < -epsilon THEN
                pivotCol := j;
                BREAK;
            END_IF;
        END_FOR;

        IF pivotCol = -1 THEN
            done := TRUE;
            CONTINUE;
        END_IF;

        // Step 3: Find leaving variable (minimum positive ratio test)
        pivotRow := -1;
        minRatio := 9999999.0;
        FOR i := 1 TO m DO
            IF tableau[i, pivotCol] > epsilon THEN
                ratio := tableau[i, m + n] / tableau[i, pivotCol];
                IF ratio < minRatio THEN
                    minRatio := ratio;
                    pivotRow := i;
                END_IF;
            END_IF;
        END_FOR;

        IF pivotRow = -1 THEN
            // Unbounded solution
            Status := 2;
            done := TRUE;
            CONTINUE;
        END_IF;

        // Step 4: Pivot operation
        pivotVal := tableau[pivotRow, pivotCol];

        // Normalize pivot row
        FOR j := 0 TO m + n DO
            tableau[pivotRow, j] := tableau[pivotRow, j] / pivotVal;
        END_FOR;

        // Eliminate other rows
        FOR i := 0 TO m DO
            IF i <> pivotRow THEN
                pivotVal := tableau[i, pivotCol];
                FOR j := 0 TO m + n DO
                    tableau[i, j] := tableau[i, j] - pivotVal * tableau[pivotRow, j];
                END_FOR;
            END_IF;
        END_FOR;

    END_WHILE;

    // Step 5: Extract solution from tableau
    FOR j := 0 TO n - 1 DO
        X[j + 1] := 0.0;
        FOR i := 1 TO m DO
            IF ABS(tableau[i, j] - 1.0) < epsilon THEN
                X[j + 1] := tableau[i, m + n];
                BREAK;
            END_IF;
        END_FOR;
    END_FOR;

    OptimalValue := -tableau[0, m + n];

    // Set status based on result
    IF pivotCol = -1 THEN
        Status := 0; // Solution found
    END_IF;

ELSE
    // Disable resets output
    FOR i := 1 TO 5 DO
        X[i] := 0.0;
    END_FOR;
    OptimalValue := 0.0;
    Status := 0;
END_IF;

PROGRAM PLC_PRG
VAR
    SimplexSolver: FB_SimplexSolver;

    // Maximize Z = 3x + 5y
    // Subject to:
    //  x ≤ 4
    //  2y ≤ 12
    // 3x + 2y ≤ 18

    C: ARRAY[1..2] OF REAL := [3.0, 5.0];
    A: ARRAY[1..3, 1..2] OF REAL := [[1.0, 0.0], [0.0, 2.0], [3.0, 2.0]];
    B: ARRAY[1..3] OF REAL := [4.0, 12.0, 18.0];
    X: ARRAY[1..2] OF REAL := [0.0, 0.0];
    Z: REAL := 0.0;
    SolutionFound: BOOL := FALSE;
END_VAR

SimplexSolver(
    C := C,
    A := A,
    B := B,
    NumVars := 2,
    NumCons := 3,
    Enable := TRUE
);

X := SimplexSolver.X;
Z := SimplexSolver.OptimalValue;
SolutionFound := SimplexSolver.Status = 0;
