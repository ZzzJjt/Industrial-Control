FUNCTION_BLOCK PoissonProbability
VAR_INPUT
    Lambda : REAL; // Expected mean number of events (λ)
    K : INT;       // Observed number of events (k)
END_VAR
VAR_OUTPUT
    Probability : REAL; // Result of the Poisson probability mass function
END_VAR
VAR
    i : INT;
    Factorial : REAL;
END_VAR

// Compute P(k; λ) = (e^(-λ) * λ^k) / k!
// Handle valid input only
IF (Lambda > 0.0) AND (K >= 0) THEN
    Factorial := 1.0;

    // Compute k! iteratively to avoid recursion
    FOR i := 1 TO K DO
        Factorial := Factorial * REAL(i);
    END_FOR;

    // Apply the Poisson formula
    Probability := EXP(-Lambda) * POW(Lambda, K) / Factorial;
ELSE
    // Invalid input: return 0 probability
    Probability := 0.0;
END_IF;
