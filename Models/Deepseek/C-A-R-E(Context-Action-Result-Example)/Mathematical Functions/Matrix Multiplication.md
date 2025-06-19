FUNCTION_BLOCK MatrixMultiply4x4
VAR_INPUT
    MatrixA : ARRAY[1..4, 1..4] OF REAL; // First input matrix
    MatrixB : ARRAY[1..4, 1..4] OF REAL; // Second input matrix
END_VAR

VAR_OUTPUT
    MatrixC : ARRAY[1..4, 1..4] OF REAL; // Resultant matrix after multiplication
END_VAR

VAR
    i, j, k : INT; // Loop indices
END_VAR

// Initialize the result matrix to zero
FOR i := 1 TO 4 DO
    FOR j := 1 TO 4 DO
        MatrixC[i,j] := 0.0;
    END_FOR;
END_FOR;

// Perform matrix multiplication
FOR i := 1 TO 4 DO
    FOR j := 1 TO 4 DO
        FOR k := 1 TO 4 DO
            MatrixC[i,j] := MatrixC[i,j] + MatrixA[i,k] * MatrixB[k,j];
        END_FOR;
    END_FOR;
END_FOR;



