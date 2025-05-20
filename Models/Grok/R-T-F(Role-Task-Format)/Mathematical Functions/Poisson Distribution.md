FUNCTION_BLOCK PoissonProbability
VAR_INPUT
    Lambda : REAL; (* Expected mean number of events (λ) *)
    K : INT; (* Number of actual events observed *)
END_VAR
VAR_OUTPUT
    Probability : REAL; (* Poisson probability P(k; λ) *)
    ErrorCode : INT := 0; (* 0: Success, 1: Invalid input *)
END_VAR
VAR
    i : INT; (* Loop index for factorial *)
    Factorial : REAL := 1.0; (* Accumulator for k! *)
    ExpLambda : REAL; (* e^(-λ) *)
    LambdaPowerK : REAL; (* λ^k *)
END_VAR

(* Initialize outputs *)
Probability := 0.0;
ErrorCode := 0;
Factorial := 1.0;

(* Validate inputs *)
IF NOT IS_VALID_REAL(Lambda) OR Lambda <= 0.0 OR K < 0 THEN
    (* Poisson PMF requires λ > 0 and k >= 0 *)
    Probability := 0.0; (* Return safe default *)
    ErrorCode := 1; (* Indicate invalid input *)
    RETURN;
END_IF;

(* Compute factorial: k! *)
IF K > 0 THEN
    FOR i := 1 TO K DO
        Factorial := Factorial * INT_TO_REAL(i);
        (* Check for overflow or non-finite result *)
        IF NOT IS_VALID_REAL(Factorial) THEN
            Probability := 0.0;
            ErrorCode := 1;
            RETURN;
        END_IF;
    END_FOR;
END_IF;

(* Compute e^(-λ) using built-in EXP function *)
ExpLambda := EXP(-Lambda);
IF NOT IS_VALID_REAL(ExpLambda) THEN
    Probability := 0.0;
    ErrorCode := 1;
    RETURN;
END_IF;

(* Compute λ^k using built-in POW function *)
LambdaPowerK := POW(Lambda, K);
IF NOT IS_VALID_REAL(LambdaPowerK) THEN
    Probability := 0.0;
    ErrorCode := 1;
    RETURN;
END_IF;

(* Compute Poisson probability: P(k; λ) = (e^(-λ) * λ^k) / k! *)
Probability := (ExpLambda * LambdaPowerK) / Factorial;

(* Validate final result *)
IF NOT IS_VALID_REAL(Probability) OR Probability < 0.0 THEN
    Probability := 0.0;
    ErrorCode := 1;
END_IF;

(* Helper function to check if a REAL value is valid *)
METHOD IS_VALID_REAL : BOOL
VAR_INPUT
    Value : REAL;
END_VAR
IS_VALID_REAL := NOT (Value = 0.0 / 0.0) AND NOT (Value = 1.0 / 0.0);
END_METHOD

END_FUNCTION_BLOCK
