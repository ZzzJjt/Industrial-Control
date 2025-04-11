```
FUNCTION_BLOCK PoissonDistribution
VAR_INPUT
    Lambda : REAL; // Mean of the Poisson distribution
    K : INT; // Input value (number of occurrences)
VAR_OUTPUT
    Probability : REAL; // Probability of K occurrences
VAR
    FactorialCache : ARRAY [0..100] OF REAL; // Cache for factorial calculations
    Sum : REAL := 0; // Accumulator for the Poisson probability
    FactorialK : REAL := 1; // Factorial of K
    Term : REAL; // Intermediate term in the Poisson formula
    OverflowDetected : BOOL := FALSE; // Flag for detecting overflow
END_VAR

// Initialize the factorial cache
FOR i := 0 TO 100 DO
    IF i = 0 THEN
        FactorialCache[i] := 1;
    ELSE
        FactorialCache[i] := FactorialCache[i-1] * REAL(i);
    END_IF;
END_FOR;

// Check for valid input values
IF Lambda >= 0 AND K >= 0 THEN
    // Use cached factorials for efficiency
    IF K <= 100 THEN
        FactorialK := FactorialCache[K];
    ELSE
        // Calculate factorial for larger K values
        OverflowDetected := TRUE;
        FOR i := 1 TO K DO
            FactorialK := FactorialK * REAL(i);
            IF IS_OVERFLOW(FactorialK) THEN
                OverflowDetected := TRUE;
                EXIT;
            END_IF;
        END_FOR;
    END_IF;
    
    // Calculate the Poisson probability
    IF NOT OverflowDetected THEN
        Term := EXP(-Lambda) * POWER(Lambda, K);
        Probability := Term / FactorialK;
    ELSE
        Probability := 0; // Set to zero if overflow occurred
    END_IF;
ELSE
    Probability := 0; // Invalid input values
END_IF;

END_FUNCTION_BLOCK
```
