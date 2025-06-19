FUNCTION_BLOCK MatrixMultiply4x4
VAR_INPUT
    MatrixA : ARRAY[1..4, 1..4] OF REAL; (* First input matrix *)
    MatrixB : ARRAY[1..4, 1..4] OF REAL; (* Second input matrix *)
END_VAR
VAR_OUTPUT
    MatrixC : ARRAY[1..4, 1..4] OF REAL; (* Resulting product matrix *)
    ErrorCode : INT := 0; (* 0: Success, 1: Invalid input *)
END_VAR
VAR
    i, j, k : INT; (* Loop indices for rows, columns, and dot product *)
    Temp : REAL; (* Temporary variable for accumulating dot product *)
END_VAR

(* Initialize output matrix and error code *)
FOR i := 1 TO 4 DO
    FOR j := 1 TO 4 DO
        MatrixC[i,j] := 0.0; (* Reset output element to zero *)
    END_FOR;
END_FOR;
ErrorCode := 0;

(* Validate inputs for non-finite values *)
FOR i := 1 TO 4 DO
    FOR j := 1 TO 4 DO
        IF NOT IS_VALID_REAL(MatrixA[i,j]) OR NOT IS_VALID_REAL(MatrixB[i,j]) THEN
            ErrorCode := 1; (* Invalid input *)
            RETURN; (* Exit early on invalid input *)
        END_IF;
    END_FOR;
END_FOR;

(* Matrix multiplication: C[i,j] = sum(A[i,k] * B[k,j]) for k=1 to 4 *)
FOR i := 1 TO 4 DO (* Iterate over rows of MatrixA *)
    FOR j := 1 TO 4 DO (* Iterate over columns of MatrixB *)
        Temp := 0.0; (* Initialize accumulator for dot product *)
        FOR k := 1 TO 4 DO (* Compute dot product *)
            (* Multiply corresponding elements and add to accumulator *)
            Temp := Temp + MatrixA[i,k] * MatrixB[k,j];
        END_FOR;
        MatrixC[i,j] := Temp; (* Store result in output matrix *)
        (* Check for non-finite result *)
        IF NOT IS_VALID_REAL(MatrixC[i,j]) THEN
            ErrorCode := 1; (* Invalid result *)
            RETURN;
        END_IF;
    END_FOR;
END_FOR;

(* Helper function to check if a REAL value is valid *)
METHOD IS_VALID_REAL : BOOL
VAR_INPUT
    Value : REAL;
END_VAR
IS_VALID_REAL := NOT (Value = 0.0 / 0.0) AND NOT (Value = 1.0 / 0.0);
END_METHOD

END_FUNCTION_BLOCK
