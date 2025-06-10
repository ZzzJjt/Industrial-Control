FUNCTION_BLOCK MatrixMultiply4x4
VAR_INPUT
    // Input matrix A (4x4)
    MatrixA : ARRAY[1..4, 1..4] OF REAL;
    
    // Input matrix B (4x4)
    MatrixB : ARRAY[1..4, 1..4] OF REAL;
END_VAR

VAR_OUTPUT
    // Output matrix C = A × B (4x4)
    MatrixC : ARRAY[1..4, 1..4] OF REAL;
END_VAR

VAR
    i, j, k : INT; // Loop indices for rows and columns
END_VAR

// ---------------- Matrix Multiplication Logic ----------------

FOR i := 1 TO 4 DO // Loop over rows of MatrixA
    FOR j := 1 TO 4 DO // Loop over columns of MatrixB
        MatrixC[i, j] := 0.0; // Reset the output element before summation
        FOR k := 1 TO 4 DO // Compute dot product of row from A and column from B
            MatrixC[i, j] := MatrixC[i, j] + MatrixA[i, k] * MatrixB[k, j];
        END_FOR;
    END_FOR;
END_FOR;
