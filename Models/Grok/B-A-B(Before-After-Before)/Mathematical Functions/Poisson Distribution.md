(* Function Block: PoissonProbability
   Purpose: Computes the Poisson probability P(k; λ) for a given mean rate λ and event count k.
   Features:
   - Inputs: Lambda (REAL, mean rate of events), K (INT, number of events)
   - Output: Probability (REAL, P(k; λ)), Error (BOOL, TRUE if computation fails)
   - Formula: P(k; λ) = (e^(-λ) * λ^k) / k!
   - Uses EXP(-Lambda) for e^(-λ), POW(Lambda, K) for λ^k, iterative loop for k!
   - Handles edge cases: negative Lambda or K, large K, numerical overflow
   - Optimized for PLCs: deterministic, bounded execution, no recursion
   Mathematical Concept:
   - Poisson distribution models the probability of k events in a fixed interval
   - λ is the expected number of events; k is the observed number
   - Valid for Lambda > 0, K >= 0; undefined otherwise
   Notes:
   - Assumes 32-bit REAL arithmetic (IEEE 754 single-precision, ~7 digits precision)
   - Limits K to 20 to prevent factorial overflow (20! ≈ 2.43E18)
   - Checks for precision loss with large Lambda or small probabilities
   - Suitable for machine failure prediction, sensor trigger monitoring, error rate modeling
*)

FUNCTION_BLOCK PoissonProbability
VAR_INPUT
    Lambda : REAL;                          (* Mean rate of events (λ) *)
    K : INT;                                (* Number of events (k) *)
END_VAR
VAR_OUTPUT
    Probability : REAL;                     (* Poisson probability P(k; λ) *)
    Error : BOOL;                           (* TRUE if input invalid or computation fails *)
END_VAR
VAR
    (* Constants *)
    MaxK : INT := 20;                       (* Maximum K to prevent factorial overflow *)
    Epsilon : REAL := 0.000001;             (* Threshold for valid Lambda *)
    MaxReal : REAL := 1.0E6;                (* Threshold for overflow detection *)
    
    (* Working variables *)
    i : INT;                                (* Loop index for factorial *)
    Factorial : REAL := 1.0;                (* Accumulates k! *)
    ExpTerm : REAL;                         (* e^(-λ) *)
    PowerTerm : REAL;                       (* λ^k *)
    Temp : REAL;                            (* Temporary calculation *)
END_VAR

(* Initialize outputs *)
Probability := 0.0;
Error := FALSE;

(* Validate inputs *)
IF Lambda <= Epsilon OR K < 0 THEN
    (* Edge case: Lambda <= 0 or K < 0, Poisson undefined *)
    Probability := 0.0;
    Error := TRUE;
ELSIF K = 0 THEN
    (* Special case: K = 0, P(0; λ) = e^(-λ) *)
    ExpTerm := EXP(-Lambda);
    IF ExpTerm <> ExpTerm OR ABS(ExpTerm) > MaxReal THEN (* NaN or overflow *)
        Probability := 0.0;
        Error := TRUE;
    ELSE
        Probability := ExpTerm;
    END_IF
ELSIF K > MaxK THEN
    (* Edge case: K too large, factorial risks overflow *)
    Probability := 0.0;
    Error := TRUE;
ELSE
    (* Compute factorial: k! *)
    FOR i := 1 TO K DO
        Factorial := Factorial * REAL(i);
        IF Factorial > MaxReal THEN
            Probability := 0.0;
            Error := TRUE;
            EXIT;
        END_IF
    END_FOR
    
    IF NOT Error THEN
        (* Compute e^(-λ) *)
        ExpTerm := EXP(-Lambda);
        
        (* Compute λ^k *)
        PowerTerm := POW(Lambda, K);
        
        (* Check for numerical issues *)
        IF ExpTerm <> ExpTerm OR PowerTerm <> PowerTerm OR  (* NaN check *)
           ABS(ExpTerm) > MaxReal OR ABS(PowerTerm) > MaxReal THEN
            Probability := 0.0;
            Error := TRUE;
        ELSE
            (* Compute P(k; λ) = (e^(-λ) * λ^k) / k! *)
            Temp := ExpTerm * PowerTerm / Factorial;
            
            (* Validate result *)
            IF Temp <> Temp OR ABS(Temp) > MaxReal OR Temp < 0.0 THEN
                Probability := 0.0;
                Error := TRUE;
            ELSE
                Probability := Temp;
            END_IF
        END_IF
    END_IF
END_IF

END_FUNCTION_BLOCK
