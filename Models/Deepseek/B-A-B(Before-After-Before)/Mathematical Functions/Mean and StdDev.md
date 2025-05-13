FUNCTION_BLOCK MeanStdDevCalculator
{ SFC: Calculate Mean and Standard Deviation }
{ Author: Automation Engineer }
{ Date: 2025-05-13 }

(*
    Description:
    Computes the mean and sample standard deviation of a fixed-size array of integers.
    Designed for use in real-time PLC applications where statistical analysis is needed.

    Input:
    - InputArray: Array of 100 integers

    Output:
    - Mean: Average value of the array elements
    - StdDev: Sample standard deviation of the array elements

    Algorithm:
    1. Compute sum of all elements for mean calculation
    2. Subtract mean from each element, square it, accumulate for variance
    3. Divide accumulated squares by (N-1) and take square root for standard deviation
*)

VAR_INPUT
    InputArray : ARRAY[1..100] OF INT; // Fixed-size input array
END_VAR

VAR_OUTPUT
    Mean : REAL;      // Computed mean
    StdDev : REAL;    // Computed sample standard deviation
END_VAR

VAR
    i : INT;          // Loop index
    Sum : REAL;       // Accumulator for sum of elements
    SumSquaredDiff : REAL; // Accumulator for sum of squared differences
    Temp : REAL;      // Temporary variable for intermediate calculations
END_VAR

// Step 1: Calculate the mean
Sum := 0.0;
FOR i := 1 TO 100 DO
    Sum := Sum + REAL(InputArray[i]); // Cast INT to REAL to prevent overflow
END_FOR;
Mean := Sum / 100.0;

// Step 2: Calculate the sample standard deviation
SumSquaredDiff := 0.0;
FOR i := 1 TO 100 DO
    Temp := REAL(InputArray[i]) - Mean; // Calculate difference from mean
    SumSquaredDiff := SumSquaredDiff + (Temp * Temp); // Square and accumulate
END_FOR;
StdDev := SQRT(SumSquaredDiff / 99.0); // Divide by N-1 and take square root

// Note: Using REAL type throughout to ensure sufficient precision and avoid overflow
// Division by 99 instead of 100 ensures we're calculating the sample standard deviation
// Suitable for deterministic execution within PLC scan cycles

END_FUNCTION_BLOCK

PROGRAM PLC_PRG
VAR
    StatsCalc: MeanStdDevCalculator;
    DataPoints : ARRAY[1..100] OF INT := [/* Insert your dataset here */];
    AverageValue : REAL;
    VariabilityMeasure : REAL;
END_VAR

StatsCalc(
    InputArray := DataPoints,
    Mean => AverageValue,
    StdDev => VariabilityMeasure
);

// Now AverageValue holds the mean, and VariabilityMeasure contains the standard deviation
