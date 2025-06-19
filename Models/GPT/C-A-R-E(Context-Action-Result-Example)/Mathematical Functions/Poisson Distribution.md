FUNCTION_BLOCK PoissonProbability
VAR_INPUT
    Lambda : REAL; // Average event rate (λ)
    K      : INT;  // Observed number of events (k)
END_VAR

VAR_OUTPUT
    Probability : REAL; // Computed Poisson probability P(k; λ)
END_VAR

VAR
    Factorial    : REAL; // Holds computed k!
    PowTerm      : REAL; // Holds λ^k
    ExpTerm      : REAL; // Holds e^(-λ)
    I            : INT;  // Loop counter
END_VAR

// Step 0: Handle edge cases
IF (K < 0) OR (Lambda <= 0.0) THEN
    Probability := 0.0;
ELSE
    // Step 1: Compute e^(-λ)
    ExpTerm := EXP(-Lambda);

    // Step 2: Compute λ^k
    PowTerm := POW(Lambda, REAL(K));

    // Step 3: Compute k! using a loop
    Factorial := 1.0;
    FOR I := 1 TO K DO
        Factorial := Factorial * REAL(I);
    END_FOR;

    // Step 4: Final result
    Probability := (ExpTerm * PowTerm) / Factorial;
END_IF;
