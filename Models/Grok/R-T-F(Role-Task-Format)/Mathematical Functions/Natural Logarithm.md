FUNCTION_BLOCK NaturalLog
VAR_INPUT
    X : REAL; (* Input value for natural logarithm *)
END_VAR
VAR_OUTPUT
    LnX : REAL; (* Result of ln(X) *)
    Error : BOOL; (* TRUE if X <= 0 or input is invalid *)
END_VAR

(* Initialize outputs *)
LnX := 0.0;
Error := FALSE;

(* Validate input *)
IF NOT IS_VALID_REAL(X) OR X <= 0.0 THEN
    (* Natural logarithm is undefined for X <= 0 or non-finite inputs *)
    LnX := 0.0; (* Return safe default *)
    Error := TRUE; (* Indicate invalid input *)
ELSE
    (* Compute natural logarithm using built-in LN() function *)
    (* ln(X) is the power to which e (≈2.71828) must be raised to obtain X *)
    LnX := LN(X);
    
    (* Check for valid result to catch any computational errors *)
    IF NOT IS_VALID_REAL(LnX) THEN
        LnX := 0.0; (* Fallback for unexpected errors *)
        Error := TRUE;
    END_IF;
END_IF;

(* Helper function to check if a REAL value is valid *)
METHOD IS_VALID_REAL : BOOL
VAR_INPUT
    Value : REAL;
END_VAR
IS_VALID_REAL := NOT (Value = 0.0 / 0.0) AND NOT (Value = 1.0 / 0.0);
END_METHOD

END_FUNCTION_BLOCK
