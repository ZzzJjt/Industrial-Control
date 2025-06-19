FUNCTION_BLOCK PoissonProbability
VAR_INPUT
    Lambda : REAL; // λ: mean rate of occurrence
    K      : INT;  // k: number of observed events
END_VAR

VAR_OUTPUT
    Probability : REAL; // P(k; λ): probability of observing k events
END_VAR

VAR
    i        : INT;
    Factorial: REAL := 1.0;
END_VAR

// Error check: K must be ≥ 0 and Lambda > 0
IF (K < 0) OR (Lambda <= 0.0) THEN
    Probability := 0.0;
ELSE
    // Compute K! iteratively
    Factorial := 1.0;
    FOR i := 1 TO K DO
        Factorial := Factorial * i;
    END_FOR;

    // Apply the Poisson formula:
    // P(k; λ) = (e^(-λ) * λ^k) / k!
    Probability := EXP(-Lambda) * POW(Lambda, K) / Factorial;
END_IF;
