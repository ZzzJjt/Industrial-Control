FUNCTION_BLOCK FB_PoissonProbability
VAR_INPUT
    Lambda : REAL := 0.0; // Expected number of occurrences (mean rate)
    K        : INT  := 0;  // Number of actual events observed
END_VAR

VAR_OUTPUT
    Probability : REAL := 0.0; // Calculated Poisson probability
END_VAR

VAR
    Factorial : REAL := 1.0;
    i         : INT  := 0;
    ExpLambda : REAL := 0.0;
    PowerTerm : REAL := 0.0;
END_VAR

// Reset internal variables at start of execution
Factorial := 1.0;

// Validate input domain: Lambda must be positive, K must be non-negative
IF (Lambda <= 0.0) OR (K < 0) THEN
    Probability := 0.0;
    EXIT; // Exit early if invalid input
END_IF;

// Step 1: Compute factorial of K manually (no built-in FACTORIAL function in ST)
FOR i := 1 TO K DO
    Factorial := Factorial * REAL(i);
END_FOR;

// Step 2: Calculate e^(-λ) using EXP()
ExpLambda := EXP(-Lambda);

// Step 3: Compute λ^k using POW() function
PowerTerm := POW(Lambda, REAL(K));

// Step 4: Combine terms to compute final Poisson probability
Probability := ExpLambda * PowerTerm / Factorial;

PROGRAM PLC_PRG
VAR
    PoissonCalc : FB_PoissonProbability;

    MeanRate   : REAL := 5.0;     // Average number of events expected
    Observed   : INT  := 3;       // Actual number of events recorded
    ProbResult : REAL := 0.0;
END_VAR

// Compute Poisson probability
PoissonCalc(
    Lambda := MeanRate,
    K := Observed
);

ProbResult := PoissonCalc.Probability;
