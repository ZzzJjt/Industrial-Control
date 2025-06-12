FUNCTION_BLOCK MatrixMultiply4x4
VAR_INPUT
    MatrixA : ARRAY[1..4, 1..4] OF REAL; // First input 4x4 matrix
    MatrixB : ARRAY[1..4, 1..4] OF REAL; // Second input 4x4 matrix
END_VAR

VAR_OUTPUT
    MatrixC : ARRAY[1..4, 1..4] OF REAL; // Resulting 4x4 matrix product
END_VAR

VAR
    i, j, k : INT; // Loop indices
END_VAR

METHOD Execute : BOOL
BEGIN
    // Initialize the output matrix to zero
    FOR i := 1 TO 4 DO
        FOR j := 1 TO 4 DO
            MatrixC[i, j] := 0.0;
        END_FOR;
    END_FOR;

    // Perform matrix multiplication using triple nested loops
    // Outer loop iterates over rows of MatrixA and MatrixC
    FOR i := 1 TO 4 DO
        // Middle loop iterates over columns of MatrixB and MatrixC
        FOR j := 1 TO 4 DO
            // Inner loop computes the dot product of row i of MatrixA and column j of MatrixB
            FOR k := 1 TO 4 DO
                MatrixC[i, j] := MatrixC[i, j] + MatrixA[i, k] * MatrixB[k, j];
            END_FOR;
        END_FOR;
    END_FOR;

    RETURN TRUE;
END_METHOD

END_FUNCTION_BLOCK
