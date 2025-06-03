FUNCTION_BLOCK MeanStdDevCalculator
VAR_INPUT
    InputArray : ARRAY[1..100] OF INT; // Input array of 100 integer values
END_VAR

VAR_OUTPUT
    Mean   : REAL; // Arithmetic mean of the array
    StdDev : REAL; // Sample standard deviation of the array
END_VAR

VAR
    i              : INT;   // Loop counter
    Sum            : REAL;  // Sum of all array values
    Temp           : REAL;  // Temporary value for (x_i - mean)
    SumSquaredDiff : REAL;  // Sum of squared differences
END_VAR

// Step 1: Calculate the mean of InputArray
Sum := 0.0;
FOR i := 1 TO 100 DO
    Sum := Sum + REAL(InputArray[i]); // Convert INT to REAL for precision
END_FOR;
Mean := Sum / 100.0; // Mean = Sum / N

// Step 2: Calculate the sum of squared deviations
SumSquaredDiff := 0.0;
FOR i := 1 TO 100 DO
    Temp := REAL(InputArray[i]) - Mean;
    SumSquaredDiff := SumSquaredDiff + (Temp * Temp);
END_FOR;

// Step 3: Compute sample standard deviation (divide by N-1)
StdDev := SQRT(SumSquaredDiff / 99.0); // N-1 = 99
