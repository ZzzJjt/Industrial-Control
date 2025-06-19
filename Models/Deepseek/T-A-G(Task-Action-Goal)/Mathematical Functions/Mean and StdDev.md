FUNCTION_BLOCK MeanStdDevCalculator
VAR_INPUT
    InputArray : ARRAY[1..100] OF INT; // Input array of 100 integers
END_VAR

VAR_OUTPUT
    Mean : REAL; // Calculated mean of the input array
    StdDev : REAL; // Calculated standard deviation of the input array
END_VAR

METHOD Execute : BOOL
BEGIN
    VAR
        i : INT;
        Sum : REAL; // Accumulator for summing input values
        SumOfSquares : REAL; // Accumulator for summing squared differences
        TempDiff : REAL; // Temporary variable for difference calculation
    END_VAR;

    // Initialize accumulators to zero
    Sum := 0.0;
    SumOfSquares := 0.0;

    // Calculate the mean
    // Convert each input value to REAL to prevent integer overflow
    FOR i := 1 TO 100 DO
        Sum := Sum + REAL(InputArray[i]);
    END_FOR;
    Mean := Sum / 100.0; // Divide by 100.0 to get the average

    // Calculate the standard deviation (sample standard deviation)
    // Loop through the array again to calculate the sum of squared differences
    FOR i := 1 TO 100 DO
        TempDiff := REAL(InputArray[i]) - Mean;
        SumOfSquares := SumOfSquares + TempDiff * TempDiff;
    END_FOR;
    StdDev := SQRT(SumOfSquares / 99.0); // Divide by 99.0 (N-1) for sample standard deviation

    RETURN TRUE;
END_METHOD

// Main execution loop
Execute();



