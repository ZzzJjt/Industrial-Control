FUNCTION_BLOCK MatrixMultiply4x4
{ SFC: Multiply two 4x4 matrices using standard matrix multiplication }
{ Author: Automation Engineer }
{ Date: 2025-05-13 }

(*
    Description:
    Performs matrix multiplication of two 4x4 matrices (MatrixA * MatrixB)
    Result stored in MatrixC.

    Standard Matrix Multiplication Formula:
    C[i,j] = Σ (k=1 to 4) A[i,k] * B[k,j]

    Complexity:
    O(n³) — For n x n matrices, this requires n³ multiplications/additions.
    For 4x4 matrices, 64 operations per call.

    Notes:
    - Designed for fixed-size 4x4 matrices for maximum efficiency
    - Assumes row-major order indexing
    - Safe for use in real-time scan cycles with proper timing budgeting
*)

VAR_INPUT
    MatrixA : ARRAY[1..4, 1..4] OF REAL; // First input matrix
    MatrixB : ARRAY[1..4, 1..4] OF REAL; // Second input matrix
END_VAR

VAR_OUTPUT
    MatrixC : ARRAY[1..4, 1..4] OF REAL; // Result matrix
END_VAR

VAR
    i, j, k : INT; // Loop indices
    sum : REAL;    // Accumulator for dot product
END_VAR

// Outer loop over rows of MatrixA
FOR i := 1 TO 4 DO
    // Inner loop over columns of MatrixB
    FOR j := 1 TO 4 DO
        // Initialize result cell
        sum := 0.0;

        // Dot product of row i of MatrixA and column j of MatrixB
        FOR k := 1 TO 4 DO
            sum := sum + MatrixA[i, k] * MatrixB[k, j]; // Multiply & accumulate
        END_FOR;

        // Store final value in output matrix
        MatrixC[i, j] := sum;
    END_FOR;
END_FOR;

END_FUNCTION_BLOCK


PROGRAM PLC_PRG
VAR
    MatrixMult: MatrixMultiply4x4;

    // Define input matrices
    MatA : ARRAY[1..4, 1..4] OF REAL :=
        [[1.0, 0.0, 0.0, 0.0],
         [0.0, 1.0, 0.0, 0.0],
         [0.0, 0.0, 1.0, 0.0],
         [0.0, 0.0, 0.0, 1.0]];

    MatB : ARRAY[1..4, 1..4] OF REAL :=
        [[2.0, 0.0, 0.0, 0.0],
         [0.0, 2.0, 0.0, 0.0],
         [0.0, 0.0, 2.0, 0.0],
         [0.0, 0.0, 0.0, 2.0]];

    ResultMat : ARRAY[1..4, 1..4] OF REAL;
END_VAR

// Call the function block
MatrixMult(
    MatrixA := MatA,
    MatrixB := MatB,
    MatrixC => ResultMat
);

// ResultMat will now contain the product of MatA and MatB
