FUNCTION_BLOCK PoissonProbability
VAR_INPUT
    Lambda : REAL; // Average number of expected events (λ)
    K : INT;       // Number of observed events
END_VAR

VAR_OUTPUT
    Probability : REAL; // Result of the Poisson distribution calculation
END_VAR

VAR
    Factorial : REAL; // Computed factorial of K
    ExponentialTerm : REAL; // e^(-Lambda)
    PowerTerm : REAL; // Lambda^K
    i : INT; // Loop index
END_VAR

// Initialize outputs
Probability := 0.0;
Factorial := 1.0;
ExponentialTerm := 0.0;
PowerTerm := 0.0;

// Handle edge cases
IF K < 0 OR Lambda <= 0.0 THEN
    Probability := 0.0;
    RETURN;
END_IF;

// Step 1: Calculate the exponential component e^(-Lambda)
ExponentialTerm := EXP(-Lambda);

// Step 2: Raise Lambda to the power of K
PowerTerm := POW(Lambda, K);

// Step 3: Compute K!
FOR i := 1 TO K DO
    Factorial := Factorial * i;
END_FOR;

// Step 4: Combine all terms to get the result
Probability := (ExponentialTerm * PowerTerm) / Factorial;

// Inline comments explaining the logic:
// The Poisson formula is P(k; λ) = (e^(-λ) * λ^k) / k!
// The factorial is computed manually using a loop to avoid recursion and ensure fixed execution time.
// Edge cases are handled by returning a probability of 0.0 if K < 0 or Lambda ≤ 0.
// Using REAL types helps minimize overflow and ensures precision.



