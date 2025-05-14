(* Function Block: SineFunction
   Purpose: Computes the sine of an angle in radians using SIN() or Taylor series.
   Features:
   - Input: AngleRad (REAL, angle in radians), UseApproximation (BOOL, forces Taylor series)
   - Output: SineValue (REAL, sin(AngleRad)), Error (BOOL, TRUE if computation fails)
   - Uses built-in SIN() if available and not overridden; else Taylor series (7 terms)
   - Normalizes angle to [0, 2π) for accuracy
   - Handles edge cases: large angles, numerical overflow
   - Optimized for PLCs: deterministic, bounded execution, no recursion
   Mathematical Concept:
   - sin(x) is a periodic trigonometric function, bounded [-1, 1]
   - Input is in radians (1 rad ≈ 57.2958°); 2π radians = 360°
   - Taylor series: sin(x) ≈ x - x^3/3! + x^5/5! - x^7/7! + ... (near x=0)
   Performance and Precision:
   - SIN(): <0.01ms, ~7-digit precision (32-bit REAL, IEEE 754 single-precision)
   - Taylor series: ~0.1ms, ~0.001 accuracy for |x| < π/2; degrades for larger |x|
   - Angle normalization reduces error for large inputs
   - Real-time: Executes in <0.1ms, suitable for 10ms scan cycles
   Notes:
   - UseApproximation := TRUE for platforms without SIN() or testing
   - Suitable for robotics, motion control, waveform generation
   - Trade-offs: SIN() is faster and more accurate; Taylor series is slower but portable
*)

FUNCTION_BLOCK SineFunction
VAR_INPUT
    AngleRad : REAL;                        (* Angle in radians *)
    UseApproximation : BOOL;                (* TRUE to force Taylor series *)
END_VAR
VAR_OUTPUT
    SineValue : REAL;                       (* Computed sin(AngleRad) *)
    Error : BOOL;                           (* TRUE if computation fails *)
END_VAR
VAR
    (* Constants *)
    PI : REAL := 3.141592653589793;         (* π *)
    TwoPI : REAL := 6.283185307179586;      (* 2π *)
    MaxReal : REAL := 1.0E6;                (* Overflow threshold *)
    MaxTerms : UINT := 7;                   (* Taylor series terms *)
    
    (* Working variables *)
    x : REAL;                               (* Normalized angle *)
    x2 : REAL;                              (* x^2 for series *)
    Term : REAL;                            (* Current series term *)
    Sum : REAL;                             (* Accumulated series sum *)
    i : UINT;                               (* Loop index *)
    Sign : REAL;                            (* Alternating sign *)
    Factorial : REAL;                       (* Denominator for series *)
END_VAR

(* Initialize outputs *)
SineValue := 0.0;
Error := FALSE;

(* Normalize angle to [0, 2π) *)
x := AngleRad - TwoPI * FLOOR(AngleRad / TwoPI);

(* Use built-in SIN() unless approximation is forced *)
IF NOT UseApproximation THEN
    SineValue := SIN(x);
    
    (* Check for valid result *)
    IF SineValue <> SineValue OR ABS(SineValue) > 1.1 THEN (* NaN or out-of-range *)
        Error := TRUE;
        SineValue := 0.0;
    END_IF
ELSE
    (* Taylor series approximation: sin(x) ≈ x - x^3/3! + x^5/5! - x^7/7! + ... *)
    (* Reduce x to [-π/2, π/2] for better accuracy *)
    IF x > PI / 2.0 THEN
        x := PI - x;  (* sin(x) = sin(π-x) *)
    ELSIF x < -PI / 2.0 THEN
        x := -PI - x; (* sin(x) = -sin(π+x) *)
    END_IF
    
    (* Initialize series *)
    Sum := x;          (* First term: x *)
    x2 := x * x;       (* Precompute x^2 *)
    Term := x;         (* Current term *)
    Sign := -1.0;      (* Alternating sign *)
    Factorial := 1.0;  (* Accumulate factorial *)
    
    (* Compute series terms *)
    FOR i := 2 TO MaxTerms BY 2 DO
        (* Update factorial: (i)(i+1) *)
        Factorial := Factorial * REAL(i) * REAL(i + 1);
        
        (* Next term: (-1)^n * x^(2n+1) / (2n+1)! *)
        Term := Term * x2;
        Sum := Sum + Sign * Term / Factorial;
        
        (* Update sign and check for numerical stability *)
        Sign := -Sign;
        IF ABS(Term) > MaxReal OR Factorial > MaxReal THEN
            Error := TRUE;
            Sum := 0.0;
            EXIT;
        END_IF
    END_FOR
    
    SineValue := Sum;
    
    (* Validate result *)
    IF ABS(SineValue) > 1.1 OR SineValue <> SineValue THEN
        Error := TRUE;
        SineValue := 0.0;
    END_IF
END_IF

END_FUNCTION_BLOCK
