(* IEC 61131-3 Structured Text: PoissonProbability Function Block *)
(* Purpose: Computes Poisson probability P(k; λ) = (e^(-λ) * λ^k) / k! *)

FUNCTION_BLOCK PoissonProbability
VAR_INPUT
    Lambda : REAL;                  (* Average number of events, λ, e.g., 4.0 *)
    K : INT;                        (* Number of observed events, k, e.g., 2 *)
END_VAR
VAR_OUTPUT
    Probability : REAL;             (* Poisson probability, P(k; λ) *)
    ErrorFlag : BOOL;               (* TRUE if invalid input or computation error *)
    DiagLog : ARRAY[1..50] OF STRING[80]; (* Diagnostic logs *)
    LogCount : INT;                 (* Number of log entries *)
END_VAR
VAR
    ExpTerm : REAL;                 (* Exponential component, e^(-λ) *)
    PowerTerm : REAL;               (* Power component, λ^k *)
    Factorial : REAL;               (* Factorial, k! *)
    i : INT;                        (* Loop index for factorial *)
    Temp : REAL;                    (* Temporary variable for validation *)
    Timestamp : STRING[20];         (* Simulated timestamp *)
    LogBufferFull : BOOL;           (* TRUE if log buffer full *)
END_VAR

(* Main Logic *)
(* Step 1: Initialize outputs *)
Probability := 0.0;
ErrorFlag := FALSE;
Factorial := 1.0;

(* Step 2: Validate inputs *)
(* Mathematical basis: Poisson probability P(k; λ) = (e^(-λ) * λ^k) / k! *)
(* - λ > 0: Average event rate must be positive *)
(* - k ≥ 0: Number of events must be non-negative *)
(* - k! computed manually, as IEC 61131-3 lacks built-in factorial *)
(* Risk: Invalid inputs (λ ≤ 0, k < 0, NaN, infinity) cause undefined behavior *)
IF NOT IS_VALID_REAL(Lambda) THEN
    ErrorFlag := TRUE;
    Probability := 0.0;
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-17 20:35:00'; (* Replace with system clock *)
        DiagLog[LogCount] := CONCAT(Timestamp, ' Invalid Input: Lambda is NaN or Infinity');
    END_IF;
    RETURN;
END_IF;

IF Lambda <= 0.0 THEN
    ErrorFlag := TRUE;
    Probability := 0.0;
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-17 20:35:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Invalid Input: Lambda=', TO_STRING(Lambda));
    END_IF;
    RETURN;
END_IF;

IF K < 0 THEN
    ErrorFlag := TRUE;
    Probability := 0.0;
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-17 20:35:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Invalid Input: K=', TO_STRING(K));
    END_IF;
    RETURN;
END_IF;

(* Step 3: Handle special case for K = 0 *)
(* For K = 0, k! = 1, λ^0 = 1, P = e^(-λ) *)
IF K = 0 THEN
    ExpTerm := EXP(-Lambda);
    IF NOT IS_VALID_REAL(ExpTerm) THEN
        ErrorFlag := TRUE;
        Probability := 0.0;
        IF LogCount < 50 THEN
            LogCount := LogCount + 1;
            Timestamp := '2025-05-17 20:35:00';
            DiagLog[LogCount] := CONCAT(Timestamp, ' Invalid EXP(-Lambda)');
        END_IF;
        RETURN;
    END_IF;
    Probability := ExpTerm;
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-17 20:35:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Probability=', 
            CONCAT(TO_STRING(Probability), CONCAT(' for Lambda=', 
            CONCAT(TO_STRING(Lambda), CONCAT(', K=', TO_STRING(K))))));
    END_IF;
    RETURN;
END_IF;

(* Step 4: Cap K to avoid factorial overflow *)
(* Risk: k! grows rapidly (e.g., 20! ≈ 2.43E18), may overflow 32-bit REAL *)
(* Limit K ≤ 20, as 20! is manageable in single-precision REAL *)
IF K > 20 THEN
    ErrorFlag := TRUE;
    Probability := 0.0;
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-17 20:35:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Invalid Input: K=', 
            CONCAT(TO_STRING(K), ' exceeds maximum (20)'));
    END_IF;
    RETURN;
END_IF;

(* Step 5: Compute exponential term *)
ExpTerm := EXP(-Lambda);
IF NOT IS_VALID_REAL(ExpTerm) THEN
    ErrorFlag := TRUE;
    Probability := 0.0;
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-17 20:35:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Invalid EXP(-Lambda)');
    END_IF;
    RETURN;
END_IF;

(* Step 6: Compute power term *)
PowerTerm := POW(Lambda, K);
IF NOT IS_VALID_REAL(PowerTerm) THEN
    ErrorFlag := TRUE;
    Probability := 0.0;
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-17 20:35:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Invalid POW(Lambda, K)');
    END_IF;
    RETURN;
END_IF;

(* Step 7: Compute factorial manually *)
(* Factorial k! = 1 * 2 * ... * k, computed iteratively to avoid recursion *)
(* Use REAL to handle large factorials (e.g., 20! ≈ 2.43E18) *)
FOR i := 1 TO K DO
    Factorial := Factorial * TO_REAL(i);
    (* Risk: Overflow for large K; mitigated by K ≤ 20 limit *)
END_FOR;

IF NOT IS_VALID_REAL(Factorial) OR Factorial <= 0.0 THEN
    ErrorFlag := TRUE;
    Probability := 0.0;
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-17 20:35:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Invalid Factorial for K=', TO_STRING(K));
    END_IF;
    RETURN;
END_IF;

(* Step 8: Compute Poisson probability *)
(* P(k; λ) = (e^(-λ) * λ^k) / k! *)
Probability := (ExpTerm * PowerTerm) / Factorial;

(* Step 9: Validate output *)
IF NOT IS_VALID_REAL(Probability) OR Probability < 0.0 THEN
    ErrorFlag := TRUE;
    Probability := 0.0;
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-17 20:35:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Invalid Probability');
    END_IF;
    RETURN;
END_IF;

(* Step 10: Log successful computation *)
IF LogCount < 50 THEN
    LogCount := LogCount + 1;
    Timestamp := '2025-05-17 20:35:00';
    DiagLog[LogCount] := CONCAT(Timestamp, ' Probability=', 
        CONCAT(TO_STRING(Probability), CONCAT(' for Lambda=', 
        CONCAT(TO_STRING(Lambda), CONCAT(', K=', TO_STRING(K))))));
END_IF;

(* Step 11: Check log buffer overflow *)
IF LogCount >= 50 THEN
    LogBufferFull := TRUE;
END_IF;

(* Notes:
   - Purpose: Computes Poisson probability P(k; λ) = (e^(-λ) * λ^k) / k!.
   - Inputs:
     - Lambda: REAL, average event rate (λ > 0, e.g., 4.0).
     - K: INT, number of events (k ≥ 0, e.g., 2).
   - Outputs:
     - Probability: REAL, Poisson probability (e.g., 0.1465).
     - ErrorFlag: BOOL, TRUE for invalid inputs or computation errors.
     - DiagLog: ARRAY[1..50] OF STRING[80], diagnostic logs.
     - LogCount: INT, number of log entries.
   - Algorithm:
     - Steps:
       1. Validate Lambda > 0, K ≥ 0, finite values.
       2. Compute e^(-λ) using EXP(-Lambda).
       3. Compute λ^k using POW(Lambda, K).
       4. Compute k! using iterative loop (1 to K).
       5. Compute P = (e^(-λ) * λ^k) / k!.
     - Edge cases:
       - K < 0: ErrorFlag := TRUE, Probability := 0.0.
       - Lambda ≤ 0: ErrorFlag := TRUE, Probability := 0.0.
       - K = 0: k! = 1, λ^0 = 1, P = e^(-λ).
       - K > 20: ErrorFlag := TRUE to avoid factorial overflow.
       - Invalid inputs/outputs: ErrorFlag := TRUE, Probability := 0.0.
   - Comments:
     - Explains Poisson formula: P(k; λ) = (e^(-λ) * λ^k) / k!.
     - Details factorial: Manual loop for compatibility, capped at K=20.
     - Covers edge cases: Validates inputs, handles overflow, logs errors.
   - Optimization:
     - Simple logic (~30–100 FLOPs, <0.1 ms on 1 MFLOP/s PLC).
     - Fixed loop depth (K ≤ 20), no recursion, deterministic execution.
     - Minimal memory (~4 KB for logs, scalars for terms).
   - Numerical Stability:
     - Uses REAL for calculations to avoid INT overflow.
     - Validates inputs/outputs for finite values.
     - Caps K at 20 to prevent factorial overflow (20! ≈ 2.43E18).
   - Safety Checks:
     - Handles invalid Lambda, K, or results with ErrorFlag, Probability=0.0.
     - Logs errors for traceability.
   - Usage:
     - Conveyor belt: Computes P(2; 4.0) ≈ 0.1465 for 2 parts/min, λ=4.
   - Platform Notes:
     - Compatible with TwinCAT, CODESYS, Siemens TIA Portal.
     - Assumes STRING[80], REAL (32-bit), INT, EXP, POW; adjust for platform limits.
     - Timestamp is static; replace with system clock (e.g., GET_SYSTEM_TIME).
     - Log buffer size (50) is practical; adjustable for memory constraints.
*)
END_FUNCTION_BLOCK
