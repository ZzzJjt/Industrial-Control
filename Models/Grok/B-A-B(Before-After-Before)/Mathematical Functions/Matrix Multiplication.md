(* Function Block: MatrixMultiply4x4
   Purpose: Multiplies two 4x4 matrices to produce a resulting 4x4 matrix.
   Features:
   - Inputs: MatrixA, MatrixB (4x4 arrays of REAL)
   - Output: MatrixC (4x4 array of REAL, result of MatrixA * MatrixB)
   - Uses standard triple nested FOR loops for matrix multiplication
   - Includes detailed comments for each operation
   - Optimized for PLCs: deterministic, minimal memory, no external libraries
   Notes:
   - Assumes 32-bit REAL arithmetic (IEEE 754 single-precision)
   - Suitable for motion control, robotics, and coordinate transformations
   - Computational complexity: O(n³) for n=4, i.e., 64 multiply-add operations
*)

FUNCTION_BLOCK MatrixMultiply4x4
VAR_INPUT
    MatrixA : ARRAY[1..4,1..4] OF REAL;     (* First input matrix *)
    MatrixB : ARRAY[1..4,1..4] OF REAL;     (* Second input matrix *)
END_VAR
VAR_OUTPUT
    MatrixC : ARRAY[1..4,1..4] OF REAL;     (* Result matrix *)
    ErrorFlag : BOOL;                       (* TRUE if computation fails *)
END_VAR
VAR
    i, j, k : UINT;                         (* Loop indices *)
    Temp : REAL;                            (* Temporary sum for each element *)
END_VAR

(* Initialize outputs *)
FOR i := 1 TO 4 DO
    FOR j := 1 TO 4 DO
        MatrixC[i,j] := 0.0;
    END_FOR
END_FOR
ErrorFlag := FALSE;

(* Perform matrix multiplication *)
FOR i := 1 TO 4 DO
    (* Iterate over rows of MatrixA *)
    FOR j := 1 TO 4 DO
        (* Iterate over columns of MatrixB *)
        Temp := 0.0;
        FOR k := 1 TO 4 DO
            (* Compute MatrixC[i,j] = sum(MatrixA[i,k] * MatrixB[k,j]) *)
            Temp := Temp + MatrixA[i,k] * MatrixB[k,j];
        END_FOR
        (* Assign result to MatrixC[i,j] *)
        MatrixC[i,j] := Temp;
        
        (* Check for numerical overflow *)
        IF ABS(MatrixC[i,j]) > 1.0E6 THEN
            ErrorFlag := TRUE;
            MatrixC[i,j] := 0.0;
        END_IF
    END_FOR
END_FOR

END_FUNCTION_BLOCK
