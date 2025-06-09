FUNCTION_BLOCK MatrixMultiply4x4
VAR_INPUT
    MatrixA : ARRAY[1..4, 1..4] OF REAL; // First 4x4 matrix input
    MatrixB : ARRAY[1..4, 1..4] OF REAL; // Second 4x4 matrix input
END_VAR

VAR_OUTPUT
    MatrixC : ARRAY[1..4, 1..4] OF REAL; // Output matrix product A × B
END_VAR

VAR
    i, j, k : INT; // Loop indices
    TempSum : REAL; // Temporary accumulator for inner product
END_VAR

// Matrix multiplication using three nested loops
// For each row i of MatrixA and column j of MatrixB:
FOR i := 1 TO 4 DO
    FOR j := 1 TO 4 DO
        TempSum := 0.0; // Reset sum for MatrixC[i,j]
        
        // Compute dot product of row i (MatrixA) and column j (MatrixB)
        FOR k := 1 TO 4 DO
            TempSum := TempSum + MatrixA[i, k] * MatrixB[k, j];
        END_FOR;
        
        // Assign computed value to result matrix
        MatrixC[i, j] := TempSum;
    END_FOR;
END_FOR;
