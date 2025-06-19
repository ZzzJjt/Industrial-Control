FUNCTION_BLOCK PoissonProbability
{ SFC: Compute Poisson Probability P(k; λ) = e^(-λ) * λ^k / k! }
{ Author: Automation Engineer }
{ Date: 2025-05-13 }

(*
    Description:
    Computes the Poisson probability mass function for given Lambda and K.
    Designed for use in real-time PLC logic requiring statistical event modeling.

    Formula:
    P(k; λ) = e^(-λ) * λ^k / k!

    Inputs:
    - Lambda: Mean rate of occurrence (must be ≥ 0)
    - K: Number of observed events (must be ≥ 0)

    Output:
    - Probability: Computed Poisson probability value (in [0, 1])

    Notes:
    - Uses iterative factorial to prevent integer overflow
    - Supports only non-negative integers for K
    - For large K or large Lambda, precision may degrade due to floating-point limits
*)

VAR_INPUT
    Lambda : REAL; // Mean rate of occurrence
    K : INT;       // Number of occurrences
END_VAR

VAR_OUTPUT
    Probability : REAL; // Resulting Poisson probability
END_VAR

VAR
    i : INT;            // Loop index
    Factorial : REAL := 1.0; // Iterative factorial accumulator
    LambdaToK : REAL;   // Lambda raised to power K
    ExpNegLambda : REAL; // e^(-Lambda)
END_VAR

// Step 1: Input validation
IF (Lambda < 0.0) OR (K < 0) THEN
    Probability := 0.0;
    RETURN; // Invalid input: Lambda or K is negative
END_IF;

// Step 2: Handle special case when k = 0
IF K = 0 THEN
    Probability := EXP(-Lambda);
    RETURN;
END_IF;

// Step 3: Compute Lambda^K using built-in POWER function
LambdaToK := POWER(Lambda, K);

// Step 4: Compute factorial of K iteratively
Factorial := 1.0;
FOR i := 1 TO K DO
    Factorial := Factorial * i;
END_FOR;

// Step 5: Compute e^(-Lambda)
ExpNegLambda := EXP(-Lambda);

// Step 6: Apply Poisson formula
Probability := (ExpNegLambda * LambdaToK) / Factorial;

// Note: All intermediate values stored in REAL to maintain numerical stability
// Avoid recursion or dynamic memory allocation for deterministic execution

END_FUNCTION_BLOCK

PROGRAM PLC_PRG
VAR
    PoissonCalc: PoissonProbability;
    AvgFailuresPerHour : REAL := 2.5; // Lambda
    ObservedFailures : INT := 3;      // K
    FailureProbability : REAL;
END_VAR

PoissonCalc(
    Lambda := AvgFailuresPerHour,
    K := ObservedFailures,
    Probability => FailureProbability
);

// Now FailureProbability contains the Poisson probability P(3; 2.5)
