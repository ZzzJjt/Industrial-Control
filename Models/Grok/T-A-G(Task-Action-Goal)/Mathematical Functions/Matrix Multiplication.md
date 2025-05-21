(* Function Block: 4x4 Matrix Multiplication *)
FUNCTION_BLOCK FB_MatrixMultiply4x4
VAR_INPUT
    Execute : BOOL;                       (* Trigger matrix multiplication *)
    MatrixA : ARRAY[1..4, 1..4] OF REAL;  (* First input matrix (4x4) *)
    MatrixB : ARRAY[1..4, 1..4] OF REAL;  (* Second input matrix (4x4) *)
END_VAR

VAR_OUTPUT
    MatrixC : ARRAY[1..4, 1..4] OF REAL;  (* Result matrix (4x4, product of A x B) *)
    Done : BOOL;                          (* Computation completed *)
    Error : BOOL;                         (* Error flag *)
    ErrorID : DWORD;                      (* Error code *)
END_VAR

VAR
    i : UINT;                             (* Row index for MatrixA *)
    j : UINT;                             (* Column index for MatrixB *)
    k : UINT;                             (* Index for dot product summation *)
    Temp : REAL;                          (* Temporary accumulator for dot product *)
END_VAR

(* Initialize outputs *)
Done := FALSE;
Error := FALSE;
ErrorID := 0;
FOR i := 1 TO 4 DO
    FOR j := 1 TO 4 DO
        MatrixC[i,j] := 0.0;              (* Initialize result matrix to zero *)
    END_FOR;
END_FOR;

(* Main logic *)
IF Execute THEN
    (* Validate inputs to prevent numerical overflow *)
    FOR i := 1 TO 4 DO
        FOR j := 1 TO 4 DO
            IF ABS(MatrixA[i,j]) > 1.0E10 OR ABS(MatrixB[i,j]) > 1.0E10 THEN
                Error := TRUE;
                ErrorID := 16#80010000;   (* Invalid input values *)
                Done := TRUE;
                RETURN;
            END_IF;
        END_FOR;
    END_FOR;

    (* Perform matrix multiplication *)
    FOR i := 1 TO 4 DO                    (* Iterate over rows of MatrixA *)
        FOR j := 1 TO 4 DO                (* Iterate over columns of MatrixB *)
            Temp := 0.0;                  (* Initialize accumulator for MatrixC[i,j] *)
            FOR k := 1 TO 4 DO            (* Compute dot product *)
                Temp := Temp + MatrixA[i,k] * MatrixB[k,j]; (* Sum A[i,k] * B[k,j] *)
            END_FOR;
            MatrixC[i,j] := Temp;         (* Store result in MatrixC *)
            
            (* Check for numerical overflow in result *)
            IF ABS(MatrixC[i,j]) > 1.0E10 THEN
                Error := TRUE;
                ErrorID := 16#80020000;   (* Numerical overflow *)
                Done := TRUE;
                RETURN;
            END_IF;
        END_FOR;
    END_FOR;

    Done := TRUE;                         (* Mark computation as complete *)
ELSE
    (* Reset outputs when Execute is FALSE *)
    Done := FALSE;
    Error := FALSE;
    ErrorID := 0;
    FOR i := 1 TO 4 DO
        FOR j := 1 TO 4 DO
            MatrixC[i,j] := 0.0;          (* Clear result matrix *)
        END_FOR;
    END_FOR;
END_IF;

END_FUNCTION_BLOCK
