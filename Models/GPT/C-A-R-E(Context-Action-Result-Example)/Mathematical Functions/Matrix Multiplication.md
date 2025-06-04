FUNCTION_BLOCK MatrixMultiply4x4
VAR_INPUT
    MatrixA : ARRAY[1..4, 1..4] OF REAL; // Input matrix A
    MatrixB : ARRAY[1..4, 1..4] OF REAL; // Input matrix B
END_VAR

VAR_OUTPUT
    MatrixC : ARRAY[1..4, 1..4] OF REAL; // Result matrix C = A * B
END_VAR

VAR
    i, j, k : INT;
    Sum : REAL;
END_VAR

// Matrix multiplication: MatrixC[i,j] = SUM(MatrixA[i,k] * MatrixB[k,j])
FOR i := 1 TO 4 DO
    FOR j := 1 TO 4 DO
        Sum := 0.0;
        FOR k := 1 TO 4 DO
            // Accumulate the product of row i from A and column j from B
            Sum := Sum + MatrixA[i, k] * MatrixB[k, j];
        END_FOR;
        // Assign computed value to MatrixC
        MatrixC[i, j] := Sum;
    END_FOR;
END_FOR;
