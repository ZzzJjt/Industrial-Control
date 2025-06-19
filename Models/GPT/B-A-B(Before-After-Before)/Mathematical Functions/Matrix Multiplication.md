FUNCTION_BLOCK MatrixMultiply4x4
VAR_INPUT
    MatrixA : ARRAY[1..4, 1..4] OF REAL; // First input matrix
    MatrixB : ARRAY[1..4, 1..4] OF REAL; // Second input matrix
END_VAR

VAR_OUTPUT
    MatrixC : ARRAY[1..4, 1..4] OF REAL; // Resultant matrix (A * B)
END_VAR

VAR
    i, j, k : INT; // Loop counters
    Sum     : REAL;
END_VAR

// Matrix multiplication: MatrixC[i,j] = Σ MatrixA[i,k] * MatrixB[k,j]
FOR i := 1 TO 4 DO
    FOR j := 1 TO 4 DO
        Sum := 0.0;
        FOR k := 1 TO 4 DO
            // Multiply element from row i of MatrixA and column j of MatrixB
            Sum := Sum + MatrixA[i,k] * MatrixB[k,j];
        END_FOR;
        // Store the result in MatrixC
        MatrixC[i,j] := Sum;
    END_FOR;
END_FOR;
