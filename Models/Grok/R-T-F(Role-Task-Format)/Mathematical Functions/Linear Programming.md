FUNCTION_BLOCK SimplexSolver
VAR_INPUT
    C : ARRAY[1..5] OF REAL; (* Objective function coefficients *)
    A : ARRAY[1..5, 1..5] OF REAL; (* Constraint matrix *)
    B : ARRAY[1..5] OF REAL; (* Right-hand side of constraints *)
    M : INT; (* Number of constraints, max 5 *)
    N : INT; (* Number of variables, max 5 *)
END_VAR
VAR_OUTPUT
    X : ARRAY[1..5] OF REAL; (* Optimal solution *)
    ObjectiveValue : REAL; (* Value of optimized objective function *)
    Status : INT; (* 0: Optimal, 1: Infeasible, 2: Unbounded, 3: Max Iterations *)
END_VAR
VAR
    (* Tableau: [A | B | C | 0] with slack variables *)
    Tableau : ARRAY[1..6, 1..11] OF REAL; (* 5 constraints + objective, 5 vars + 5 slack + 1 RHS *)
    PivotCol, PivotRow : INT; (* Pivot column and row indices *)
    PivotValue, MinRatio, Temp : REAL; (* Temporary variables for pivot operations *)
    i, j, k, Iter : INT; (* Loop indices and iteration counter *)
    MaxIterations : INT := 50; (* Limit to prevent infinite loops *)
    IsOptimal : BOOL; (* Flag for convergence *)
    Epsilon : REAL := 1.0E-6; (* Small value for numerical stability *)
END_VAR

(* Initialize outputs *)
FOR i := 1 TO 5 DO
    X[i] := 0.0;
END_FOR;
ObjectiveValue := 0.0;
Status := 0;

(* Input validation *)
IF M > 5 OR N > 5 OR M < 1 OR N < 1 THEN
    Status := 1; (* Infeasible due to invalid dimensions *)
    RETURN;
END_IF;
FOR i := 1 TO M DO
    IF NOT IS_VALID_REAL(B[i]) OR B[i] < 0.0 THEN
        Status := 1; (* Infeasible: negative RHS *)
        RETURN;
    END_IF;
    FOR j := 1 TO N DO
        IF NOT IS_VALID_REAL(A[i,j]) THEN
            Status := 1; (* Invalid constraint matrix *)
            RETURN;
        END_IF;
    END_FOR;
END_FOR;
FOR j := 1 TO N DO
    IF NOT IS_VALID_REAL(C[j]) THEN
        Status := 1; (* Invalid objective coefficients *)
        RETURN;
    END_IF;
END_FOR;

(* Initialize tableau *)
FOR i := 1 TO M DO
    FOR j := 1 TO N DO
        Tableau[i,j] := A[i,j]; (* Copy constraint matrix *)
    END_FOR;
    FOR j := N+1 TO N+M DO
        Tableau[i,j] := 0.0; (* Initialize slack variables *)
        IF j = N+i THEN
            Tableau[i,j] := 1.0; (* Set slack variable for constraint i *)
        END_IF;
    END_FOR;
    Tableau[i,N+M+1] := B[i]; (* Set RHS *)
END_FOR;
FOR j := 1 TO N DO
    Tableau[M+1,j] := -C[j]; (* Objective function (negated for maximization) *)
END_FOR;
FOR j := N+1 TO N+M DO
    Tableau[M+1,j] := 0.0; (* Slack variables in objective row *)
END_FOR;
Tableau[M+1,N+M+1] := 0.0; (* Objective value *)

(* Simplex algorithm main loop *)
Iter := 0;
IsOptimal := FALSE;
WHILE Iter < MaxIterations AND NOT IsOptimal DO
    (* Find pivot column: most negative coefficient in objective row *)
    PivotCol := 0;
    MinRatio := 0.0;
    FOR j := 1 TO N+M DO
        IF Tableau[M+1,j] < -Epsilon AND (PivotCol = 0 OR Tableau[M+1,j] < MinRatio) THEN
            PivotCol := j;
            MinRatio := Tableau[M+1,j];
        END_IF;
    END_FOR;
    
    (* Check for optimality or unboundedness *)
    IF PivotCol = 0 THEN
        IsOptimal := TRUE; (* No negative coefficients, optimal *)
        EXIT;
    END_IF;
    
    (* Find pivot row: minimum positive ratio test *)
    PivotRow := 0;
    MinRatio := 1.0E10;
    FOR i := 1 TO M DO
        IF Tableau[i,PivotCol] > Epsilon THEN
            Temp := Tableau[i,N+M+1] / Tableau[i,PivotCol];
            IF Temp >= 0.0 AND Temp < MinRatio THEN
                MinRatio := Temp;
                PivotRow := i;
            END_IF;
        END_IF;
    END_FOR;
    
    (* Check for unboundedness *)
    IF PivotRow = 0 THEN
        Status := 2; (* Unbounded *)
        RETURN;
    END_IF;
    
    (* Pivot operation: normalize pivot row *)
    PivotValue := Tableau[PivotRow,PivotCol];
    FOR j := 1 TO N+M+1 DO
        Tableau[PivotRow,j] := Tableau[PivotRow,j] / PivotValue;
    END_FOR;
    
    (* Update other rows *)
    FOR i := 1 TO M+1 DO
        IF i <> PivotRow THEN
            Temp := Tableau[i,PivotCol];
            FOR j := 1 TO N+M+1 DO
                Tableau[i,j] := Tableau[i,j] - Temp * Tableau[PivotRow,j];
            END_FOR;
        END_IF;
    END_FOR;
    
    Iter := Iter + 1;
END_WHILE;

(* Check for non-convergence *)
IF Iter >= MaxIterations THEN
    Status := 3; (* Max iterations reached *)
    RETURN;
END_IF;

(* Extract solution *)
FOR j := 1 TO N DO
    X[j] := 0.0;
    FOR i := 1 TO M DO
        IF ABS(Tableau[i,j] - 1.0) < Epsilon AND ABS(Tableau[i,j]) > Epsilon THEN
            X[j] := Tableau[i,N+M+1]; (* Basic variable value *)
        END_IF;
    END_FOR;
END_FOR;
ObjectiveValue := -Tableau[M+1,N+M+1]; (* Objective value, negated back *)

(* Helper function to check if a REAL value is valid *)
METHOD IS_VALID_REAL : BOOL
VAR_INPUT
    Value : REAL;
END_VAR
IS_VALID_REAL := NOT (Value = 0.0 / 0.0) AND NOT (Value = 1.0 / 0.0);
END_METHOD

END_FUNCTION_BLOCK
