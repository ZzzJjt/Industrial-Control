FUNCTION_BLOCK MeanStdDevCalculator
VAR_INPUT
    // Input data: 100-point integer array (e.g., from sensors or quality checks)
    InputArray : ARRAY[1..100] OF INT;
END_VAR

VAR_OUTPUT
    // Output statistical results
    Mean    : REAL; // Arithmetic mean of the values
    StdDev  : REAL; // Sample standard deviation
END_VAR

VAR
    i                : INT;   // Loop index
    Sum              : REAL;  // Accumulator for mean
    SumSquaredDiff   : REAL;  // Accumulator for squared differences
    Diff             : REAL;  // Temporary difference from mean
END_VAR

// ---------- Step 1: Compute Mean ----------
Sum := 0.0; // Initialize accumulator
FOR i := 1 TO 100 DO
    Sum := Sum + REAL(InputArray[i]); // Use REAL to avoid integer overflow
END_FOR;
Mean := Sum / 100.0; // Compute arithmetic mean

// ---------- Step 2: Compute Sample Standard Deviation ----------
SumSquaredDiff := 0.0; // Reset accumulator
FOR i := 1 TO 100 DO
    Diff := REAL(InputArray[i]) - Mean; // Difference from mean
    SumSquaredDiff := SumSquaredDiff + (Diff * Diff); // Square and accumulate
END_FOR;
StdDev := SQRT(SumSquaredDiff / 99.0); // Sample std. dev (N-1 for unbiased estimate)
