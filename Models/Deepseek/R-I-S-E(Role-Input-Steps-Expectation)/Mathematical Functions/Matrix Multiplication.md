FUNCTION_BLOCK FB_MatrixMultiply4x4
VAR_INPUT
    MatrixA : ARRAY[1..4, 1..4] OF REAL := [[0.0]]; // First input matrix
    MatrixB : ARRAY[1..4, 1..4] OF REAL := [[0.0]]; // Second input matrix
END_VAR

VAR_OUTPUT
    MatrixC : ARRAY[1..4, 1..4] OF REAL := [[0.0]]; // Resulting product matrix
END_VAR

VAR
    i, j, k : DINT; // Loop counters
    sum : REAL;     // Accumulator for dot product
END_VAR

// Multiply two 4x4 matrices using standard matrix multiplication algorithm:
// C[i,j] = Σ (k=1 to 4) A[i,k] * B[k,j]

FOR i := 1 TO 4 DO
    FOR j := 1 TO 4 DO
        sum := 0.0;

        // Compute the dot product of row i of MatrixA and column j of MatrixB
        FOR k := 1 TO 4 DO
            sum := sum + MatrixA[i, k] * MatrixB[k, j];
        END_FOR;

        // Store the result in output matrix
        MatrixC[i, j] := sum;
    END_FOR;
END_FOR;

PROGRAM PLC_PRG
VAR
    MatrixMultiplier: FB_MatrixMultiply4x4;

    // Define two 4x4 matrices
    A : ARRAY[1..4, 1..4] OF REAL :=
        [[1.0, 2.0, 3.0, 4.0],
         [5.0, 6.0, 7.0, 8.0],
         [9.0, 10.0, 11.0, 12.0],
         [13.0, 14.0, 15.0, 16.0]];

    B : ARRAY[1..4, 1..4] OF REAL :=
        [[16.0, 15.0, 14.0, 13.0],
         [12.0, 11.0, 10.0, 9.0],
         [8.0, 7.0, 6.0, 5.0],
         [4.0, 3.0, 2.0, 1.0]];

    ResultMatrix : ARRAY[1..4, 1..4] OF REAL;
END_VAR

// Perform matrix multiplication
MatrixMultiplier(
    MatrixA := A,
    MatrixB := B
);

ResultMatrix := MatrixMultiplier.MatrixC;
