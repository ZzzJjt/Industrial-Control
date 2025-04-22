```
FUNCTION_BLOCK MatrixMultiply
VAR_INPUT
    MatrixA : ARRAY [1..4, 1..4] OF REAL; // First matrix
    MatrixB : ARRAY [1..4, 1..4] OF REAL; // Second matrix
VAR_OUTPUT
    Product : ARRAY [1..4, 1..4] OF REAL; // Resultant matrix product
VAR
    i, j, k : INT; // Loop counters
END_VAR

// Initialize the product matrix with zeros
FOR i := 1 TO 4 DO
    FOR j := 1 TO 4 DO
        Product[i, j] := 0;
    END_FOR
END_FOR

// Matrix multiplication
FOR i := 1 TO 4 DO
    FOR j := 1 TO 4 DO
        FOR k := 1 TO 4 DO
            // Accumulate the product of the corresponding row and column elements
            Product[i, j] := Product[i, j] + MatrixA[i, k] * MatrixB[k, j];
        END_FOR
    END_FOR
END_FOR

END_FUNCTION_BLOCK
```
