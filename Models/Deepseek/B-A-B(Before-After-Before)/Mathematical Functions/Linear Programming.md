FUNCTION_BLOCK SimplexSolver
{ SFC: Simplex solver for linear programming }
{ Author: Automation Engineer }
{ Date: 2025-05-13 }

(*
    Description:
    Solves a linear programming problem using the Simplex algorithm.
    Supports up to 5 decision variables and 5 constraints.

    Input:
    - C: Coefficients of the objective function to maximize
    - A: Constraint coefficient matrix
    - B: Right-hand-side values of constraints
    - Maximize: Set TRUE for maximization; FALSE for minimization

    Output:
    - X: Solution vector (decision variables)
    - Z: Final objective value
    - Status: Solution status (0=Optimal, 1=Infeasible, 2=Iteration Limit Reached)
*)

VAR_INPUT
    C : ARRAY[0..4] OF REAL := [0.0, 0.0, 0.0, 0.0, 0.0]; // Objective function coefficients
    A : ARRAY[0..4, 0..4] OF REAL := [[0.0]];            // Constraint matrix
    B : ARRAY[0..4] OF REAL := [0.0];                    // RHS values
    NumVars : INT := 2;                                  // Number of variables (<=5)
    NumCons : INT := 2;                                  // Number of constraints (<=5)
    Maximize : BOOL := TRUE;                             // Optimization direction
END_VAR

VAR_OUTPUT
    X : ARRAY[0..4] OF REAL := [0.0];                    // Solution vector
    Z : REAL := 0.0;                                     // Final objective value
    Status : INT := 0;                                   // 0=Optimal, 1=Infeasible, 2=IterationLimit
END_VAR

VAR
    // Internal tableau storage (rows = constraints + 1, cols = variables + slack + artificial + 1)
    Tableau : ARRAY[0..5, 0..11] OF REAL := [[0.0]];
    i, j, k : INT;
    pivotCol, pivotRow : INT;
    minRatio, ratio : REAL;
    done : BOOL := FALSE;
    iterCount : INT := 0;
    MAX_ITER : INT := 50;

    // Helper variables
    temp : REAL;
    hasArtificial : BOOL := FALSE;
END_VAR

// Step 1: Initialize tableau
FOR i := 0 TO NumCons DO
    FOR j := 0 TO NumVars + NumCons + 1 DO
        IF j < NumVars THEN
            Tableau[i, j] := A[i, j];
        ELSIF j < NumVars + NumCons THEN
            IF i = j - NumVars THEN
                Tableau[i, j] := 1.0; // Identity matrix for slack/artificial variables
            ELSE
                Tableau[i, j] := 0.0;
            END_IF;
        ELSIF j = NumVars + NumCons THEN
            Tableau[i, j] := B[i]; // RHS
        END_IF;
    END_FOR;
END_FOR;

// Step 2: Initialize objective row
FOR j := 0 TO NumVars + NumCons DO
    IF j < NumVars THEN
        Tableau[NumCons, j] := IF Maximize THEN -C[j] ELSE C[j]; // Flip sign for maximization
    ELSE
        Tableau[NumCons, j] := 0.0;
    END_IF;
END_FOR;

Tableau[NumCons, NumVars + NumCons + 1] := 0.0; // Initial objective value

// Step 3: Main simplex loop
WHILE NOT done AND iterCount < MAX_ITER DO
    // Find pivot column (most negative coefficient in objective row)
    pivotCol := -1;
    FOR j := 0 TO NumVars + NumCons DO
        IF Tableau[NumCons, j] < -1.0E-8 THEN
            IF pivotCol = -1 OR Tableau[NumCons, j] < Tableau[NumCons, pivotCol] THEN
                pivotCol := j;
            END_IF;
        END_IF;
    END_FOR;

    IF pivotCol = -1 THEN
        done := TRUE; // All coefficients are non-negative → optimal
        Status := 0;
        CONTINUE;
    END_IF;

    // Find pivot row (minimum positive ratio)
    pivotRow := -1;
    minRatio := 999999999.0;
    FOR i := 0 TO NumCons - 1 DO
        IF Tableau[i, pivotCol] > 1.0E-8 THEN
            ratio := Tableau[i, NumVars + NumCons] / Tableau[i, pivotCol];
            IF ratio < minRatio THEN
                minRatio := ratio;
                pivotRow := i;
            END_IF;
        END_IF;
    END_FOR;

    IF pivotRow = -1 THEN
        Status := 1; // Unbounded/infeasible
        done := TRUE;
        CONTINUE;
    END_IF;

    // Normalize pivot row
    temp := Tableau[pivotRow, pivotCol];
    FOR j := 0 TO NumVars + NumCons + 1 DO
        Tableau[pivotRow, j] := Tableau[pivotRow, j] / temp;
    END_FOR;

    // Eliminate other rows
    FOR i := 0 TO NumCons DO
        IF i <> pivotRow THEN
            temp := Tableau[i, pivotCol];
            FOR j := 0 TO NumVars + NumCons + 1 DO
                Tableau[i, j] := Tableau[i, j] - temp * Tableau[pivotRow, j];
            END_FOR;
        END_IF;
    END_FOR;

    iterCount := iterCount + 1;
END_WHILE;

// Step 4: Check if max iterations reached
IF NOT done THEN
    Status := 2;
END_IF;

// Step 5: Extract solution from tableau
FOR j := 0 TO NumVars DO
    X[j] := 0.0;
END_FOR;

FOR i := 0 TO NumCons - 1 DO
    FOR j := 0 TO NumVars DO
        IF ABS(Tableau[i, j] - 1.0) < 1.0E-6 THEN
            X[j] := Tableau[i, NumVars + NumCons];
        END_IF;
    END_FOR;
END_FOR;

// Return final objective value
Z := Tableau[NumCons, NumVars + NumCons + 1];

END_FUNCTION_BLOCK

PROGRAM PLC_PRG
VAR
    Simplex1: SimplexSolver;
    C : ARRAY[0..1] OF REAL := [3.0, 5.0]; // Maximize Z = 3x + 5y
    A : ARRAY[0..1, 0..1] OF REAL := [[1.0, 0.0], [0.0, 2.0], [3.0, 2.0]]; // Constraints
    B : ARRAY[0..1] OF REAL := [4.0, 12.0, 18.0]; // RHS
    Solution : ARRAY[0..1] OF REAL;
    ObjectiveValue : REAL;
END_VAR

Simplex1(
    C := C,
    A := A,
    B := B,
    NumVars := 2,
    NumCons := 2,
    Maximize := TRUE
);

Solution := Simplex1.X;
ObjectiveValue := Simplex1.Z;
