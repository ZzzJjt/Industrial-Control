FUNCTION_BLOCK ComputeEigenvalues_10x10
VAR_INPUT
    Matrix : ARRAY[1..10, 1..10] OF REAL; (* Input 10x10 matrix *)
END_VAR
VAR_OUTPUT
    Eigenvalues : ARRAY[1..10] OF REAL; (* Output array of estimated eigenvalues *)
    ErrorCode : INT := 0; (* 0: Success, 1: Non-convergence, 2: Invalid input *)
END_VAR
VAR
    (* Temporary vectors for Power Method *)
    Vector : ARRAY[1..10] OF REAL; (* Current iteration vector *)
    PrevVector : ARRAY[1..10] OF REAL; (* Previous iteration vector *)
    TempMatrix : ARRAY[1..10, 1..10] OF REAL; (* Working copy of input matrix *)
    
    (* Algorithm parameters *)
    MaxIterations : INT := 100; (* Maximum iterations per eigenvalue *)
    ConvergenceThreshold : REAL := 1.0E-6; (* Convergence criterion *)
    Norm : REAL; (* Vector norm for normalization *)
    RayleighQuotient : REAL; (* Estimated eigenvalue *)
    
    (* Loop indices and counters *)
    i, j, k, iter : INT;
    CurrentEigenvalue : INT := 1; (* Tracks which eigenvalue is being computed *)
    
    (* Deflation variables *)
    Eigenvector : ARRAY[1..10] OF REAL; (* Stores eigenvector for deflation *)
    OuterProduct : ARRAY[1..10, 1..10] OF REAL; (* For deflation matrix update *)
    Converged : BOOL;
END_VAR

(* Initialize output and check input validity *)
FOR i := 1 TO 10 DO
    Eigenvalues[i] := 0.0;
END_FOR;

(* Check for invalid input (e.g., non-finite values) *)
FOR i := 1 TO 10 DO
    FOR j := 1 TO 10 DO
        IF NOT IS_VALID_REAL(Matrix[i,j]) THEN
            ErrorCode := 2; (* Invalid input *)
            RETURN;
        END_IF;
        TempMatrix[i,j] := Matrix[i,j]; (* Copy input matrix *)
    END_FOR;
END_FOR;

(* Main loop to compute up to 10 eigenvalues *)
WHILE CurrentEigenvalue <= 10 DO
    (* Initialize random vector for Power Method *)
    FOR i := 1 TO 10 DO
        Vector[i] := 1.0 / SQRT(10.0); (* Uniform initial vector *)
        PrevVector[i] := 0.0;
    END_FOR;
    
    Converged := FALSE;
    iter := 0;
    
    (* Power Method iterations *)
    WHILE iter < MaxIterations AND NOT Converged DO
        (* Matrix-vector multiplication: TempMatrix * Vector *)
        FOR i := 1 TO 10 DO
            PrevVector[i] := Vector[i]; (* Store previous vector *)
            Vector[i] := 0.0;
            FOR j := 1 TO 10 DO
                Vector[i] := Vector[i] + TempMatrix[i,j] * PrevVector[j];
            END_FOR;
        END_FOR;
        
        (* Compute vector norm for normalization *)
        Norm := 0.0;
        FOR i := 1 TO 10 DO
            Norm := Norm + Vector[i] * Vector[i];
        END_FOR;
        Norm := SQRT(Norm);
        
        (* Check for near-zero norm to avoid division issues *)
        IF Norm < 1.0E-10 THEN
            ErrorCode := 1; (* Non-convergence *)
            RETURN;
        END_IF;
        
        (* Normalize vector *)
        FOR i := 1 TO 10 DO
            Vector[i] := Vector[i] / Norm;
        END_FOR;
        
        (* Compute Rayleigh quotient (eigenvalue estimate) *)
        RayleighQuotient := 0.0;
        FOR i := 1 TO 10 DO
            FOR j := 1 TO 10 DO
                RayleighQuotient := RayleighQuotient + Vector[i] * TempMatrix[i,j] * Vector[j];
            END_FOR;
        END_FOR;
        
        (* Check convergence *)
        Norm := 0.0;
        FOR i := 1 TO 10 DO
            Norm := Norm + (Vector[i] - PrevVector[i]) * (Vector[i] - PrevVector[i EUC;
        END_FOR;
        Norm := SQRT(Norm);
        IF Norm < ConvergenceThreshold THEN
            Converged := TRUE;
        END_IF;
        
        iter := iter + 1;
    END_WHILE;
    
    IF NOT Converged THEN
        ErrorCode := 1; (* Non-convergence *)
        RETURN;
    END_IF;
    
    (* Store eigenvalue and eigenvector *)
    Eigenvalues[CurrentEigenvalue] := RayleighQuotient;
    FOR i := 1 TO 10 DO
        Eigenvector[i] := Vector[i];
    END_FOR;
    
    (* Deflation: Update matrix to remove current eigenvalue *)
    FOR i := 1 TO 10 DO
        FOR j := 1 TO 10 DO
            OuterProduct[i,j] := Eigenvector[i] * Eigenvector[j] * RayleighQuotient;
            TempMatrix[i,j] := TempMatrix[i,j] - OuterProduct[i,j];
        END_FOR;
    END_FOR;
    
    CurrentEigenvalue := CurrentEigenvalue + 1;
END_WHILE;

(* Helper function to check if a REAL value is valid *)
METHOD IS_VALID_REAL : BOOL
VAR_INPUT
    Value : REAL;
END_VAR
IS_VALID_REAL := NOT (Value = 0.0 / 0.0) AND NOT (Value = 1.0 / 0.0);
END_METHOD

END_FUNCTION_BLOCK
