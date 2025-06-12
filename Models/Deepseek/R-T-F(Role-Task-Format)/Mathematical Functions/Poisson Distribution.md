FUNCTION_BLOCK PoissonProbability
VAR_INPUT
    Lambda : REAL; // Expected mean number of events (λ)
    K : INT;       // Number of actual events observed
END_VAR

VAR_OUTPUT
    Probability : REAL; // Calculated Poisson probability
END_VAR

VAR
    i : INT;          // Loop index for factorial calculation
    Factorial : REAL := 1.0; // Accumulator for factorial computation
    ExpLambda : REAL; // Precomputed value of e^(-λ)
    LambdaK : REAL;   // Precomputed value of λ^K
END_VAR

METHOD Execute : BOOL
BEGIN
    // Initialize output to zero
    Probability := 0.0;

    // Check for valid inputs: Lambda > 0 and K >= 0
    IF (Lambda > 0.0) AND (K >= 0) THEN
        // Compute the factorial of K iteratively
        FOR i := 1 TO K DO
            Factorial := Factorial * i;
        END_FOR;

        // Precompute e^(-λ) using the EXP() function
        ExpLambda := EXP(-Lambda);

        // Precompute λ^K using the POW() function
        LambdaK := POW(Lambda, K);

        // Calculate the Poisson probability
        Probability := ExpLambda * LambdaK / Factorial;
    ELSE
        // Set Probability to 0.0 for invalid inputs
        Probability := 0.0;
    END_IF;

    RETURN TRUE;
END_METHOD

END_FUNCTION_BLOCK
