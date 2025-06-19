(* Function Block: NaturalLog
   Purpose: Computes the natural logarithm ln(X) for a given REAL input X.
   Features:
   - Input: X (REAL, value to compute logarithm of)
   - Outputs: LnX (REAL, computed ln(X)), Error (BOOL, TRUE if input invalid)
   - Uses built-in LN() function if available, else Taylor series for x near 1
   - Handles edge cases: x <= 0 (Error := TRUE), x = 1 (LnX := 0.0)
   - Optimized for PLCs: deterministic, bounded execution, no recursion
   Mathematical Concept:
   - ln(x) is the natural logarithm, the inverse of e^x, where e ≈ 2.71828
   - Defined only for x > 0; undefined for x <= 0
   - For x = 1, ln(1) = 0
   Notes:
   - Assumes 32-bit REAL arithmetic (IEEE 754 single-precision, ~7 digits precision)
   - Taylor series (if used) is centered at x=1, with 10 terms for ~0.001 accuracy
   - Suitable for flowmeter linearization, sensor calibration, exponential decay modeling
*)

FUNCTION_BLOCK NaturalLog
VAR_INPUT
    X : REAL;                               (* Input value for ln(X) *)
END_VAR
VAR_OUTPUT
    LnX : REAL;                             (* Computed natural logarithm *)
    Error : BOOL;                           (* TRUE if input is invalid (x <= 0) *)
END_VAR
VAR
    (* Constants for Taylor series approximation *)
    MaxTerms : UINT := 10;                  (* Number of terms for series *)
    Epsilon : REAL := 0.000001;             (* Threshold for valid input *)
    
    (* Working variables for approximation *)
    i : UINT;                               (* Loop index *)
    Term : REAL;                            (* Current term in series *)
    Sum : REAL;                             (* Accumulated series sum *)
    Y : REAL;                               (* Reduced input: (x-1)/x *)
END_VAR

(* Initialize outputs *)
LnX := 0.0;
Error := FALSE;

(* Check input domain: x > 0 *)
IF X <= Epsilon THEN
    (* Edge case: x <= 0, logarithm undefined *)
    LnX := 0.0;
    Error := TRUE;
ELSIF ABS(X - 1.0) < Epsilon THEN
    (* Edge case: x = 1, ln(1) = 0 *)
    LnX := 0.0;
    Error := FALSE;
ELSE
    (* Attempt to use built-in LN function *)
    LnX := LN(X);  (* Assumes IEC 61131-3 compliant LN() *)
    
    (* Check for valid result *)
    IF LnX <> LnX OR ABS(LnX) > 1.0E6 THEN (* NaN or overflow check *)
        (* Fallback to Taylor series for x near 1 *)
        IF ABS(X - 1.0) < 0.5 THEN
            (* Taylor series: ln(x) ≈ Σ[(-1)^(n+1) * ((x-1)/x)^n / n] for x near 1 *)
            Y := (X - 1.0) / X;  (* Range reduction *)
            Sum := 0.0;
            Term := Y;           (* First term *)
            
            FOR i := 1 TO MaxTerms DO
                (* Compute term: ((-1)^(n+1) * Y^n) / n *)
                Sum := Sum + Term / REAL(i);
                Term := -Term * Y;  (* Next term: alternate sign, multiply by Y *)
                
                (* Check for numerical stability *)
                IF ABS(Term) > 1.0E6 THEN
                    Error := TRUE;
                    Sum := 0.0;
                    EXIT;
                END_IF
            END_FOR
            LnX := Sum;
        ELSE
            (* Input too far from 1 or LN() failed *)
            LnX := 0.0;
            Error := TRUE;
        END_IF
    END_IF
END_IF

(* Final overflow check *)
IF ABS(LnX) > 1.0E6 AND NOT Error THEN
    LnX := 0.0;
    Error := TRUE;
END_IF

END_FUNCTION_BLOCK
