(* IEC 61131-3 Structured Text: LinearInterpolation Function Block *)
(* Purpose: Performs linear interpolation between two points *)

FUNCTION_BLOCK LinearInterpolation
VAR_INPUT
    X1 : REAL;                      (* X-coordinate of first known point, e.g., 0.5 V *)
    Y1 : REAL;                      (* Y-coordinate of first known point, e.g., 0°C *)
    X2 : REAL;                      (* X-coordinate of second known point, e.g., 2.5 V *)
    Y2 : REAL;                      (* Y-coordinate of second known point, e.g., 100°C *)
    X : REAL;                       (* Query point X for interpolation, e.g., 1.5 V *)
END_VAR
VAR_OUTPUT
    Y : REAL;                       (* Interpolated Y value, e.g., 50°C *)
    ErrorFlag : BOOL;               (* TRUE if computation fails, e.g., X1 = X2 *)
    DiagLog : ARRAY[1..50] OF STRING[80]; (* Diagnostic logs *)
    LogCount : INT;                 (* Number of log entries *)
END_VAR
VAR
    DeltaX : REAL;                  (* X2 - X1 for denominator *)
    DeltaY : REAL;                  (* Y2 - Y1 for numerator *)
    Slope : REAL;                   (* (Y2 - Y1) / (X2 - X1) *)
    Timestamp : STRING[20];         (* Simulated timestamp *)
    LogBufferFull : BOOL;           (* TRUE if log buffer full *)
END_VAR

(* Main Logic *)
(* Step 1: Initialize outputs *)
Y := 0.0;
ErrorFlag := FALSE;

(* Step 2: Validate inputs for finite values *)
IF NOT IS_VALID_REAL(X1) OR NOT IS_VALID_REAL(Y1) OR 
   NOT IS_VALID_REAL(X2) OR NOT IS_VALID_REAL(Y2) OR 
   NOT IS_VALID_REAL(X) THEN
    ErrorFlag := TRUE;
    Y := Y1; (* Fallback to Y1 *)
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-17 20:11:00'; (* Replace with system clock *)
        DiagLog[LogCount] := CONCAT(Timestamp, ' Invalid Input Values');
    END_IF;
    RETURN;
END_IF;

(* Step 3: Compute differences *)
DeltaX := X2 - X1;
DeltaY := Y2 - Y1;

(* Step 4: Check for zero denominator to avoid division by zero *)
IF ABS(DeltaX) < 1E-6 THEN (* Consider X1 ≈ X2 *)
    ErrorFlag := TRUE;
    Y := Y1; (* Fallback to Y1 *)
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-17 20:11:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Division by Zero: X1 ≈ X2');
    END_IF;
    RETURN;
END_IF;

(* Step 5: Compute slope *)
Slope := DeltaY / DeltaX;

(* Step 6: Perform linear interpolation *)
(* Formula: Y = Y1 + ((X - X1) * (Y2 - Y1)) / (X2 - X1) *)
Y := Y1 + (X - X1) * Slope;

(* Step 7: Validate output *)
IF NOT IS_VALID_REAL(Y) THEN
    ErrorFlag := TRUE;
    Y := Y1; (* Fallback to Y1 *)
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-17 20:11:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Invalid Output Y');
    END_IF;
    RETURN;
END_IF;

(* Step 8: Log successful interpolation *)
IF LogCount < 50 THEN
    LogCount := LogCount + 1;
    Timestamp := '2025-05-17 20:11:00';
    DiagLog[LogCount] := CONCAT(Timestamp, ' Interpolation: X=', 
        CONCAT(TO_STRING(X), CONCAT(', Y=', TO_STRING(Y))));
END_IF;

(* Step 9: Check log buffer overflow *)
IF LogCount >= 50 THEN
    LogBufferFull := TRUE;
END_IF;

(* Notes:
   - Purpose: Performs linear interpolation between points (X1,Y1) and (X2,Y2) for query X.
   - Inputs:
     - X1, Y1: REAL, coordinates of first point (e.g., 0.5 V, 0°C).
     - X2, Y2: REAL, coordinates of second point (e.g., 2.5 V, 100°C).
     - X: REAL, query point (e.g., 1.5 V).
   - Outputs:
     - Y: REAL, interpolated value (e.g., 50°C).
     - ErrorFlag: BOOL, TRUE for invalid inputs or X1 ≈ X2.
     - DiagLog: ARRAY[1..50] OF STRING[80], diagnostic logs.
     - LogCount: INT, number of log entries.
   - Formula:
     - Y = Y1 + ((X - X1) * (Y2 - Y1)) / (X2 - X1)
     - If X1 ≈ X2, returns Y1 and sets ErrorFlag.
   - Edge Cases:
     - X1 = X2: Sets ErrorFlag, returns Y1, logs error.
     - Invalid inputs (NaN, infinity): Sets ErrorFlag, returns Y1, logs error.
   - Optimization:
     - Single formula evaluation (~10 FLOPs, <0.1 ms on 1 MFLOP/s PLC).
     - No loops, deterministic execution for real-time (10–100 ms cycles).
     - Minimal memory (~4 KB for DiagLog).
   - Numerical Stability:
     - Checks for X1 ≈ X2 (ABS(DeltaX) < 1E-6) to avoid division by zero.
     - Validates finite inputs to prevent overflow/NaN.
     - Stable formula minimizes rounding errors in 32-bit REAL.
   - Safety Checks:
     - Validates inputs and output for finite values.
     - Returns Y1 as fallback on errors.
     - Logs errors for traceability.
   - Usage:
     - Temperature sensor: Interpolates X=1.5 V to Y=50°C for (0.5 V, 0°C), (2.5 V, 100°C).
   - Platform Notes:
     - Assumes STRING[80], REAL (32-bit); adjust for platform limits.
     - Timestamp is static; replace with system clock (e.g., GET_SYSTEM_TIME).
     - Compatible with PLCs (e.g., Siemens S7, CODESYS) using Profibus DP or similar.
     - Log buffer size (50) is practical; adjustable for memory constraints.
*)
END_FUNCTION_BLOCK
