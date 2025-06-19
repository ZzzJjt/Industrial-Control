(* Function Block: Natural Logarithm Calculator *)
(* Version: 1.0, Date: 2025-05-21 *)
FUNCTION_BLOCK FB_NaturalLog
VAR_INPUT
    Execute : BOOL;                   (* Trigger computation *)
    X : REAL;                         (* Input value for ln(X) *)
END_VAR

VAR_OUTPUT
    LnX : REAL;                       (* Natural logarithm result, ln(X) *)
    Done : BOOL;                      (* Computation completed *)
    Error : BOOL;                     (* Error flag: TRUE if X <= 0 *)
    ErrorID : DWORD;                  (* Diagnostic code: 0x80010001 for X <= 0 *)
END_VAR

(* Initialize outputs *)
LnX := 0.0;
Done := FALSE;
Error := FALSE;
ErrorID := 0;

(* Main logic *)
IF Execute THEN
    (* Validate input: ln(X) is undefined for X <= 0 *)
    IF X > 0.0 THEN
        (* Compute natural logarithm using built-in LN function *)
        (* LN(X) returns the natural logarithm (base e) of X *)
        LnX := LN(X);
        
        (* Check for numerical overflow or invalid result *)
        (* IEEE 754 REAL typically handles ln(X) for X in [1.0E-38, 1.0E38] *)
        IF ABS(LnX) > 1.0E10 THEN
            Error := TRUE;
            ErrorID := 16#80020000;   (* Numerical overflow in result *)
            LnX := 0.0;               (* Fallback to zero *)
        ELSE
            Error := FALSE;
            ErrorID := 0;
        END_IF;
    ELSE
        (* Handle domain error: X <= 0 is invalid for ln(X) *)
        (* Raising Error ensures safe handling in control logic *)
        Error := TRUE;
        ErrorID := 16#80010001;       (* Invalid input: X <= 0 *)
        LnX := 0.0;                   (* Fallback to zero for predictable behavior *)
    END_IF;
    
    Done := TRUE;                     (* Mark computation as complete *)
ELSE
    (* Reset outputs when Execute is FALSE *)
    (* Ensures deterministic behavior between scans *)
    LnX := 0.0;
    Done := FALSE;
    Error := FALSE;
    ErrorID := 0;
END_IF;

END_FUNCTION_BLOCK
