```
FUNCTION_BLOCK ComputeEigenvalues
VAR_INPUT
    Matrix : ARRAY [1..10, 1..10] OF REAL; // 10x10 Matrix
VAR_OUTPUT
    Eigenvalues : ARRAY [1..10] OF REAL; // Eigenvalues
VAR
    Iterations : INT := 0; // Count iterations
    MaxIterations : INT := 1000; // Maximum allowed iterations
    Tolerance : REAL := 0.0001; // Convergence tolerance
    Q : ARRAY [1..10, 1..10] OF REAL; // Orthogonal matrix Q
    R : ARRAY [1..10, 1..10] OF REAL; // Upper triangular matrix R
END_VAR

// Initial QR Decomposition
FOR i := 1 TO 10 DO
    FOR j := 1 TO 10 DO
        Q[i,j] := 0;
        R[i,j] := 0;
    END_FOR
END_FOR

FOR i := 1 TO 10 DO
    Q[i,i] := 1;
END_FOR

// QR Algorithm Loop
WHILE Iterations < MaxIterations DO
    // Perform QR Decomposition
    QRDecomposition(Matrix, Q, R);
    
    // Reconstruct Matrix from Q and R
    Matrix := MatMul(Q, R);
    
    // Check Diagonal for Convergence
    BoolArray := ARRAY [1..10] OF BOOLEAN;
    FOR i := 1 TO 9 DO
        BoolArray[i] := ABS(R[i+1,i]) < Tolerance;
    END_FOR
    BoolArray[10] := TRUE; // Last element is always converged
    
    // If all diagonal elements are converged, break loop
    IF ALL(BoolArray) THEN
        Eigenvalues := ARRAY [1..10] OF REAL;
        FOR i := 1 TO 10 DO
            Eigenvalues[i] := R[i,i];
        END_FOR
        EXIT;
    END_IF
    
    Iterations := Iterations + 1;
END_WHILE

// If max iterations reached without convergence, set Eigenvalues to NaN
IF Iterations >= MaxIterations THEN
    FOR i := 1 TO 10 DO
        Eigenvalues[i] := NaN;
    END_FOR
END_IF

// Helper Functions for QR Decomposition and Matrix Multiplication
FUNCTION QRDecomposition(Matrix, Q, R : ARRAY [1..10, 1..10] OF REAL) : BOOL;
    // Implementation of QR Decomposition Algorithm
END_FUNCTION

FUNCTION MatMul(A, B : ARRAY [1..10, 1..10] OF REAL) : ARRAY [1..10, 1..10] OF REAL;
    // Implementation of Matrix Multiplication
END_FUNCTION

// Helper Function to Check if All Elements in Array are TRUE
FUNCTION ALL(Array : ARRAY [1..10] OF BOOLEAN) : BOOLEAN;
    // Implementation of Checking All Elements
END_FUNCTION

END_FUNCTION_BLOCK
```
