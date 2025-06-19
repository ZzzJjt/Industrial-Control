FUNCTION_BLOCK MeanStdDevCalculator
VAR_INPUT
    InputArray : ARRAY[1..100] OF INT; // Input array of 100 integers
END_VAR

VAR_OUTPUT
    Mean   : REAL; // Mean value of the input array
    StdDev : REAL; // Sample standard deviation
END_VAR

VAR
    i               : INT;
    Sum             : REAL := 0.0;
    SumSquaredDiff  : REAL := 0.0;
    Temp            : REAL;
END_VAR

// Step 1: Calculate the mean
Sum := 0.0;
FOR i := 1 TO 100 DO
    Sum := Sum + REAL(InputArray[i]); // Convert INT to REAL to avoid overflow
END_FOR;
Mean := Sum / 100.0; // Compute mean

// Step 2: Calculate the sum of squared differences from the mean
SumSquaredDiff := 0.0;
FOR i := 1 TO 100 DO
    Temp := REAL(InputArray[i]) - Mean; // Difference from mean
    SumSquaredDiff := SumSquaredDiff + Temp * Temp; // Square and accumulate
END_FOR;

// Step 3: Compute standard deviation (sample)
StdDev := SQRT(SumSquaredDiff / 99.0); // Divide by N-1 = 99 for sample std dev
