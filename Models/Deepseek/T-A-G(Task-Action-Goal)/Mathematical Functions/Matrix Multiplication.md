FUNCTION_BLOCK MatrixMultiply4x4
VAR_INPUT
    MatrixA : ARRAY[1..4, 1..4] OF REAL; // First input matrix (4x4)
    MatrixB : ARRAY[1..4, 1..4] OF REAL; // Second input matrix (4x4)
END_VAR

VAR_OUTPUT
    MatrixC : ARRAY[1..4, 1..4] OF REAL; // Resultant matrix (4x4)
END_VAR

METHOD Execute : BOOL
BEGIN
    VAR
        i, j, k : INT; // Loop indices
    END_VAR;

    // Initialize the resultant matrix to zero
    // This is important to ensure that previous results do not affect the current computation
    FOR i := 1 TO 4 DO
        FOR j := 1 TO 4 DO
            MatrixC[i, j] := 0.0;
        END_FOR;
    END_FOR;

    // Perform matrix multiplication using three nested loops
    // Outer loops iterate through rows of MatrixA and columns of MatrixB
    // Inner loop performs the dot product and accumulates the result in MatrixC[i,j]
    FOR i := 1 TO 4 DO
        FOR j := 1 TO 4 DO
            FOR k := 1 TO 4 DO
                MatrixC[i, j] := MatrixC[i, j] + MatrixA[i, k] * MatrixB[k, j];
            END_FOR;
        END_FOR;
    END_FOR;

    RETURN TRUE;
END_METHOD

// Main execution loop
Execute();



