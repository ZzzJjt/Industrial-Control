(* IEC 61131-3 Structured Text: MatrixMultiply4x4 Function Block *)
(* Purpose: Performs 4x4 matrix multiplication for two input matrices *)

FUNCTION_BLOCK MatrixMultiply4x4
VAR_INPUT
    MatrixA : ARRAY[1..4, 1..4] OF REAL; (* First 4x4 input matrix *)
    MatrixB : ARRAY[1..4, 1..4] OF REAL; (* Second 4x4 input matrix *)
END_VAR
VAR_OUTPUT
    MatrixC : ARRAY[1..4, 1..4] OF REAL; (* Resulting 4x4 matrix, C = A * B *)
    ErrorFlag : BOOL;                    (* TRUE if computation fails, e.g., invalid inputs *)
    DiagLog : ARRAY[1..50] OF STRING[80]; (* Diagnostic logs *)
    LogCount : INT;                      (* Number of log entries *)
END_VAR
VAR
    i, j, k : INT;                       (* Loop indices for rows, columns, and summation *)
    Sum : REAL;                          (* Temporary sum for product terms *)
    Timestamp : STRING[20];              (* Simulated timestamp *)
    LogBufferFull : BOOL;                (* TRUE if log buffer full *)
END_VAR

(* Main Logic *)
(* Step 1: Initialize outputs *)
ErrorFlag := FALSE;
FOR i := 1 TO 4 DO
    FOR j := 1 TO 4 DO
        MatrixC[i,j] := 0.0; (* Initialize result matrix to zero *)
    END_FOR;
END_FOR;

(* Step 2: Validate inputs for finite values *)
FOR i := 1 TO 4 DO
    FOR j := 1 TO 4 DO
        IF NOT IS_VALID_REAL(MatrixA[i,j]) OR NOT IS_VALID_REAL(MatrixB[i,j]) THEN
            ErrorFlag := TRUE;
            IF LogCount < 50 THEN
                LogCount := LogCount + 1;
                Timestamp := '2025-05-17 20:11:00'; (* Replace with system clock *)
                DiagLog[LogCount] := CONCAT(Timestamp, ' Invalid Input at MatrixA[', 
                    CONCAT(TO_STRING(i), CONCAT(',', CONCAT(TO_STRING(j), '] or MatrixB'))));
            END_IF;
            RETURN;
        END_IF;
    END_FOR;
END_FOR;

(* Step 3: Perform matrix multiplication *)
(* C[i,j] = Σ (A[i,k] * B[k,j]) for k=1..4 *)
FOR i := 1 TO 4 DO (* Iterate over rows of MatrixA *)
    FOR j := 1 TO 4 DO (* Iterate over columns of MatrixB *)
        Sum := 0.0; (* Initialize sum for C[i,j] *)
        FOR k := 1 TO 4 DO (* Sum product terms *)
            (* Accumulate A[i,k] * B[k,j] *)
            Sum := Sum + MatrixA[i,k] * MatrixB[k,j];
        END_FOR;
        MatrixC[i,j] := Sum; (* Store result in C[i,j] *)
    END_FOR;
END_FOR;

(* Step 4: Validate output *)
FOR i := 1 TO 4 DO
    FOR j := 1 TO 4 DO
        IF NOT IS_VALID_REAL(MatrixC[i,j]) THEN
            ErrorFlag := TRUE;
            FOR k := 1 TO 4 DO
                FOR m := 1 TO 4 DO
                    MatrixC[k,m] := 0.0; (* Reset MatrixC on error *)
                END_FOR;
            END_FOR;
            IF LogCount < 50 THEN
                LogCount := LogCount + 1;
                Timestamp := '2025-05-17 20:11:00';
                DiagLog[LogCount] := CONCAT(Timestamp, ' Invalid Output at MatrixC[', 
                    CONCAT(TO_STRING(i), CONCAT(',', CONCAT(TO_STRING(j), ']'))));
            END_IF;
            RETURN;
        END_IF;
    END_FOR;
END_FOR;

(* Step 5: Log successful computation *)
IF LogCount < 50 THEN
    LogCount := LogCount + 1;
    Timestamp := '2025-05-17 20:11:00';
    DiagLog[LogCount] := CONCAT(Timestamp, ' Matrix Multiplication Completed');
END_IF;

(* Step 6: Check log buffer overflow *)
IF LogCount >= 50 THEN
    LogBufferFull := TRUE;
END_IF;

(* Notes:
   - Purpose: Multiplies two 4x4 matrices (C = A * B) for applications like robot kinematics.
   - Inputs:
     - MatrixA: ARRAY[1..4, 1..4] OF REAL, first input matrix.
     - MatrixB: ARRAY[1..4, 1..4] OF REAL, second input matrix.
   - Outputs:
     - MatrixC: ARRAY[1..4, 1..4] OF REAL, result matrix.
     - ErrorFlag: BOOL, TRUE for invalid inputs or output.
     - DiagLog: ARRAY[1..50] OF STRING[80], diagnostic logs.
     - LogCount: INT, number of log entries.
   - Algorithm:
     - Standard matrix multiplication: C[i,j] = Σ (A[i,k] * B[k,j]) for k=1..4.
     - Uses triple nested FOR loops for row-by-column computation.
     - Steps:
       1. Initialize MatrixC to zero.
       2. Validate MatrixA, MatrixB for finite values.
       3. Compute product terms using nested loops.
       4. Validate MatrixC for finite values.
       5. Log completion or errors.
   - Optimization:
     - Fixed 4x4 size (~192 bytes for matrices, <5 KB with logs).
     - ~512 FLOPs (~0.5 ms on 1 MFLOP/s PLC), fits 10–100 ms cycles.
     - No dynamic allocation, deterministic execution.
   - Numerical Stability:
     - Validates inputs/output for finite values to prevent overflow/NaN.
     - 32-bit REAL rounding mitigated by simple operations.
   - Safety Checks:
     - Checks MatrixA, MatrixB, MatrixC for valid REALs.
     - Returns zeroed MatrixC on errors, sets ErrorFlag.
     - Logs errors for traceability.
   - Usage:
     - Robot arm control: Multiplies 4x4 transformation matrices for end-effector pose.
     - Example: Combines rotation and translation matrices, logs completion.
   - Platform Notes:
     - Compatible with TwinCAT, CODESYS, Siemens TIA Portal.
     - Assumes STRING[80], REAL (32-bit), INT; adjust for platform limits.
     - Timestamp is static; replace with system clock (e.g., GET_SYSTEM_TIME).
     - Log buffer size (50) is practical; adjustable for memory constraints.
*)
END_FUNCTION_BLOCK
