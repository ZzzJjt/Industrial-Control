FUNCTION_BLOCK PoissonProbability
VAR_INPUT
    Lambda : REAL; // Expected number of events (λ), must be > 0
    K      : INT;  // Actual observed events, must be ≥ 0
END_VAR
VAR_OUTPUT
    Probability : REAL; // Output probability P(k; λ)
END_VAR
VAR
    i        : INT;
    Factorial : REAL := 1.0;
    Valid    : BOOL;
END_VAR

// Step 1: Validate input domain
Valid := (Lambda > 0.0) AND (K >= 0);

// Step 2: Calculate K! manually, as IEC ST has no built-in factorial function
IF Valid THEN
    Factorial := 1.0;
    FOR i := 1 TO K DO
        Factorial := Factorial * REAL(i);
    END_FOR;

    // Step 3: Apply Poisson PMF formula
    // P(k; λ) = (e^(-λ) * λ^k) / k!
    Probability := EXP(-Lambda) * POW(Lambda, K) / Factorial;
ELSE
    Probability := 0.0; // Invalid inputs return zero probability
END_IF;
