FUNCTION_BLOCK MatrixMultiply4x4
VAR_INPUT
    MatrixA : ARRAY[1..4, 1..4] OF REAL;
    MatrixB : ARRAY[1..4, 1..4] OF REAL;
END_VAR
VAR_OUTPUT
    MatrixC : ARRAY[1..4, 1..4] OF REAL;
END_VAR
VAR
    i : INT; // Row index for MatrixA
    j : INT; // Column index for MatrixB
    k : INT; // Index for dot product
END_VAR

// Matrix multiplication logic
// MatrixC[i,j] = Sum(MatrixA[i,k] * MatrixB[k,j]) for k = 1..4
FOR i := 1 TO 4 DO
    FOR j := 1 TO 4 DO
        MatrixC[i,j] := 0.0; // Initialize accumulator to zero
        FOR k := 1 TO 4 DO
            MatrixC[i,j] := MatrixC[i,j] + MatrixA[i,k] * MatrixB[k,j];
        END_FOR;
    END_FOR;
END_FOR;
