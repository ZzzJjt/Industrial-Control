FUNCTION_BLOCK MeanStdDevCalculator
VAR_INPUT
    InputArray : ARRAY[1..100] OF INT; // Array of 100 integer inputs
END_VAR

VAR_OUTPUT
    Mean : REAL; // Calculated mean of the input array
    StdDev : REAL; // Calculated sample standard deviation of the input array
END_VAR

VAR
    Sum : REAL; // Sum of all elements in the input array
    SumSquaredDiff : REAL; // Sum of squared differences from the mean
    Temp : REAL; // Temporary variable for calculations
    i : INT; // Loop index
END_VAR

// Initialize sums to zero
Sum := 0.0;
SumSquaredDiff := 0.0;

// Step 1: Calculate the sum of all elements in the input array
FOR i := 1 TO 100 DO
    Sum := Sum + InputArray[i];
END_FOR;

// Step 2: Calculate the mean by dividing the sum by 100
Mean := Sum / 100.0;

// Step 3: Calculate the sum of squared differences from the mean
FOR i := 1 TO 100 DO
    Temp := InputArray[i] - Mean;
    SumSquaredDiff := SumSquaredDiff + (Temp * Temp);
END_FOR;

// Step 4: Calculate the sample standard deviation
// Use N-1 (99) in the denominator to get the sample standard deviation
IF SumSquaredDiff >= 0.0 THEN
    StdDev := SQRT(SumSquaredDiff / 99.0);
ELSE
    StdDev := 0.0; // Handle any unexpected negative value gracefully
END_IF;

// Note: Using REAL types for internal calculations avoids overflow or precision loss.
// The use of N-1 instead of N in the denominator provides an unbiased estimate of the population variance.



