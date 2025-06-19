FUNCTION_BLOCK MeanStdDevCalculator
VAR_INPUT
    InputArray : ARRAY[1..100] OF INT; // Input array of 100 integer samples
END_VAR

VAR_OUTPUT
    Mean  : REAL; // Arithmetic average of the array
    StdDev : REAL; // Sample standard deviation of the array
END_VAR

VAR
    i         : INT;   // Loop index
    Sum       : REAL;  // Sum of elements (REAL to avoid overflow)
    SumSqDiff : REAL;  // Sum of squared differences from the mean
    TempDiff  : REAL;  // Temporary variable to hold (x_i - mean)
END_VAR

// Step 1: Compute the mean value
Sum := 0.0;

FOR i := 1 TO 100 DO
    Sum := Sum + REAL(InputArray[i]); // Cast INT to REAL for safe summation
END_FOR;

Mean := Sum / 100.0; // Compute arithmetic mean

// Step 2: Compute the sample standard deviation
SumSqDiff := 0.0;

FOR i := 1 TO 100 DO
    TempDiff := REAL(InputArray[i]) - Mean;      // Difference from mean
    SumSqDiff := SumSqDiff + TempDiff * TempDiff; // Square and accumulate
END_FOR;

StdDev := SQRT(SumSqDiff / 99.0); // Use N-1 = 99.0 for sample standard deviation
