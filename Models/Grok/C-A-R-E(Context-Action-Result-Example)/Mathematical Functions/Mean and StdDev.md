(* IEC 61131-3 Structured Text: MeanStdDevCalculator Function Block *)
(* Purpose: Computes mean and sample standard deviation of a 100-element integer array *)

FUNCTION_BLOCK MeanStdDevCalculator
VAR_INPUT
    InputArray : ARRAY[1..100] OF INT; (* Input array of 100 integers, e.g., weight measurements *)
END_VAR
VAR_OUTPUT
    Mean : REAL;                       (* Mean of the input array *)
    StdDev : REAL;                     (* Sample standard deviation, using N-1 *)
    ErrorFlag : BOOL;                  (* TRUE if computation fails, e.g., invalid input *)
    DiagLog : ARRAY[1..50] OF STRING[80]; (* Diagnostic logs *)
    LogCount : INT;                    (* Number of log entries *)
END_VAR
VAR
    Sum : REAL;                        (* Sum of array elements for mean *)
    SumSquaredDiff : REAL;             (* Sum of squared differences for standard deviation *)
    Temp : REAL;                       (* Temporary difference from mean *)
    i : INT;                           (* Loop index *)
    Timestamp : STRING[20];            (* Simulated timestamp *)
    LogBufferFull : BOOL;              (* TRUE if log buffer full *)
END_VAR

(* Main Logic *)
(* Step 1: Initialize outputs *)
Mean := 0.0;
StdDev := 0.0;
ErrorFlag := FALSE;
Sum := 0.0;
SumSquaredDiff := 0.0;

(* Step 2: Validate inputs for valid integer range *)
(* INT range: -32768 to 32767 to prevent overflow *)
FOR i := 1 TO 100 DO
    IF InputArray[i] < -32768 OR InputArray[i] > 32767 THEN
        ErrorFlag := TRUE;
        IF LogCount < 50 THEN
            LogCount := LogCount + 1;
            Timestamp := '2025-05-17 20:32:00'; (* Replace with system clock *)
            DiagLog[LogCount] := CONCAT(Timestamp, ' Invalid Input at index ', TO_STRING(i));
        END_IF;
        RETURN;
    END_IF;
END_FOR;

(* Step 3: Compute sum of array elements *)
(* Risk: Integer summation may overflow for large values; use REAL to mitigate *)
FOR i := 1 TO 100 DO
    Sum := Sum + TO_REAL(InputArray[i]);
END_FOR;

(* Step 4: Compute mean *)
(* Mean = Sum / 100, using REAL for precision *)
Mean := Sum / 100.0;

(* Step 5: Validate mean *)
IF NOT IS_VALID_REAL(Mean) THEN
    ErrorFlag := TRUE;
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-17 20:32:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Invalid Mean Calculation');
    END_IF;
    RETURN;
END_IF;

(* Step 6: Compute sum of squared differences *)
(* For each element: (InputArray[i] - Mean)^2 *)
FOR i := 1 TO 100 DO
    Temp := TO_REAL(InputArray[i]) - Mean;
    SumSquaredDiff := SumSquaredDiff + Temp * Temp;
    (* Risk: Large Temp values may cause overflow in REAL; mitigated by INT range check *)
END_FOR;

(* Step 7: Compute sample standard deviation *)
(* StdDev = sqrt((Σ (x_i - Mean)^2) / (N-1)) *)
(* Use N-1 (99) for sample standard deviation to correct bias (Bessel's correction) *)
(* N-1 is standard for process monitoring, as data is a sample of a larger population *)
IF SumSquaredDiff < 0.0 THEN (* Rare numerical error check *)
    ErrorFlag := TRUE;
    StdDev := 0.0;
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-17 20:32:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Negative SumSquaredDiff');
    END_IF;
    RETURN;
END_IF;

StdDev := SQRT(SumSquaredDiff / 99.0);

(* Step 8: Validate standard deviation *)
IF NOT IS_VALID_REAL(StdDev) THEN
    ErrorFlag := TRUE;
    StdDev := 0.0;
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-17 20:32:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Invalid StdDev Calculation');
    END_IF;
    RETURN;
END_IF;

(* Step 9: Log successful computation *)
IF LogCount < 50 THEN
    LogCount := LogCount + 1;
    Timestamp := '2025-05-17 20:32:00';
    DiagLog[LogCount] := CONCAT(Timestamp, ' Mean=', 
        CONCAT(TO_STRING(Mean), CONCAT(', StdDev=', TO_STRING(StdDev))));
END_IF;

(* Step 10: Check log buffer overflow *)
IF LogCount >= 50 THEN
    LogBufferFull := TRUE;
END_IF;

(* Notes:
   - Purpose: Computes mean and sample standard deviation of 100 INT values.
   - Input:
     - InputArray: ARRAY[1..100] OF INT, e.g., weight measurements.
   - Outputs:
     - Mean: REAL, average of array elements.
     - StdDev: REAL, sample standard deviation (N-1 denominator).
     - ErrorFlag: BOOL, TRUE for invalid inputs or computation errors.
     - DiagLog: ARRAY[1..50] OF STRING[80], diagnostic logs.
     - LogCount: INT, number of log entries.
   - Algorithm:
     - Mean: Sum all elements, divide by 100.0.
     - StdDev: Sum (x_i - Mean)^2, divide by 99.0, take square root.
   - Comments:
     - Explains steps (e.g., "Compute sum of array elements").
     - Warns risks (e.g., "Integer summation may overflow").
     - Clarifies N-1 (Bessel's correction for sample standard deviation).
   - Optimization:
     - Two single-pass loops (~1000 FLOPs, ~1 ms on 1 MFLOP/s PLC).
     - Fixed array size (100 INTs ≈ 200 bytes, logs ~4 KB, total <5 KB).
     - Deterministic execution, no dynamic allocation.
   - Numerical Stability:
     - Uses REAL for calculations to avoid INT overflow.
     - Validates inputs (-32768 to 32767) and outputs for finite values.
     - Checks negative SumSquaredDiff to catch numerical errors.
   - Safety Checks:
     - Validates input range, mean, and StdDev.
     - Returns Mean=0.0, StdDev=0.0 on errors.
     - Logs errors for traceability.
   - Usage:
     - Packaging line: Computes mean and StdDev of 100 weights, adjusts calibration, triggers alarms.
     - Example: Mean=50.2, StdDev=3.1 for weight measurements.
   - Platform Notes:
     - Compatible with TwinCAT, CODESYS, Siemens TIA Portal.
     - Assumes STRING[80], REAL (32-bit), INT; adjust for platform limits.
     - Timestamp is static; replace with system clock (e.g., GET_SYSTEM_TIME).
     - Log buffer size (50) is practical; adjustable for memory constraints.
*)
END_FUNCTION_BLOCK
