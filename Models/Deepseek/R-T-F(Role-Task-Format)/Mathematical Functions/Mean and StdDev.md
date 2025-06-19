FUNCTION_BLOCK MeanStdDevCalculator
VAR_INPUT
    InputArray : ARRAY[1..100] OF INT; // Input array of 100 integer values
END_VAR

VAR_OUTPUT
    Mean : REAL; // Calculated mean of the input array
    StdDev : REAL; // Calculated sample standard deviation of the input array
END_VAR

VAR
    Sum : REAL; // Accumulator for summing elements
    SumSquaredDiff : REAL; // Accumulator for summing squared differences
    Diff : REAL; // Difference between each element and the mean
    i : INT; // Loop index
END_VAR

METHOD Execute : BOOL
BEGIN
    // Initialize accumulators to zero
    Sum := 0.0;
    SumSquaredDiff := 0.0;

    // Calculate the sum of all elements in the input array
    FOR i := 1 TO 100 DO
        Sum := Sum + InputArray[i];
    END_FOR;

    // Calculate the mean by dividing the sum by 100.0
    Mean := Sum / 100.0;

    // Calculate the sum of squared differences from the mean
    FOR i := 1 TO 100 DO
        Diff := InputArray[i] - Mean;
        SumSquaredDiff := SumSquaredDiff + (Diff * Diff);
    END_FOR;

    // Calculate the sample standard deviation
    // Divisor is 99.0 (N-1) for sample standard deviation
    StdDev := SQRT(SumSquaredDiff / 99.0);

    RETURN TRUE;
END_METHOD

END_FUNCTION_BLOCK
