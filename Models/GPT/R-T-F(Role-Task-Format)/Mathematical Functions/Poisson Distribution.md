FUNCTION_BLOCK PoissonProbability
VAR_INPUT
    Lambda : REAL;  // Expected number of events (λ)
    K      : INT;   // Observed number of events (k)
END_VAR

VAR_OUTPUT
    Probability : REAL;  // Resulting Poisson probability P(k; λ)
END_VAR

VAR
    i        : INT;      // Loop counter
    Factorial : REAL := 1.0; // Holds the computed value of k!
END_VAR

// ---------- Poisson Probability Logic ----------
IF (Lambda > 0.0) AND (K >= 0) THEN
    // Compute factorial k! = 1 * 2 * ... * k using a loop
    Factorial := 1.0;
    FOR i := 1 TO K DO
        Factorial := Factorial * i;
    END_FOR;

    // Apply Poisson PMF formula:
    // P(k; λ) = e^(-λ) * λ^k / k!
    Probability := EXP(-Lambda) * POW(Lambda, REAL(K)) / Factorial;
ELSE
    // Handle invalid input cases gracefully
    Probability := 0.0;
END_IF;
