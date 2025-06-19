FUNCTION_BLOCK PoissonProbability
VAR_INPUT
    Lambda : REAL; // Expected mean number of events (λ)
    K : INT; // Observed number of events
END_VAR

VAR_OUTPUT
    Probability : REAL; // Result of the Poisson probability mass function
END_VAR

METHOD Execute : BOOL
BEGIN
    VAR
        i : INT;
        Factorial : REAL; // Variable to store factorial of K
    END_VAR;

    // Initialize the probability to 0.0
    Probability := 0.0;

    // Check if Lambda is greater than 0 and K is non-negative
    IF (Lambda > 0.0) AND (K >= 0) THEN
        // Initialize factorial to 1.0
        Factorial := 1.0;

        // Compute K! using a loop
        FOR i := 1 TO K DO
            Factorial := Factorial * i;
        END_FOR;

        // Calculate the Poisson probability using the formula:
        // P(k; λ) = (e^(-λ) * λ^k) / k!
        Probability := EXP(-Lambda) * POW(Lambda, K) / Factorial;
    ELSE
        // If Lambda <= 0 or K < 0, set probability to 0.0
        Probability := 0.0;
    END_IF;

    RETURN TRUE;
END_METHOD

// Main execution loop
Execute();



