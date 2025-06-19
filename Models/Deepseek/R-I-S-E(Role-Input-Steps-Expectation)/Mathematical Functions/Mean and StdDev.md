FUNCTION_BLOCK FB_MeanStdDevCalculator
VAR_INPUT
    InputArray : ARRAY[1..100] OF INT; // Input data (e.g., sensor readings, counts)
END_VAR

VAR_OUTPUT
    Mean : REAL := 0.0;     // Arithmetic mean
    StdDev : REAL := 0.0;   // Sample standard deviation (N-1)
END_VAR

VAR
    sum : REAL := 0.0;
    sumSqDiff : REAL := 0.0;
    i : DINT := 0;
    diff : REAL := 0.0;
END_VAR

// Reset outputs at start of execution
sum := 0.0;
sumSqDiff := 0.0;

// Step 1: Compute the arithmetic mean
FOR i := 1 TO 100 DO
    sum := sum + REAL(InputArray[i]); // Cast to REAL to prevent overflow
END_FOR;

Mean := sum / 100.0; // Average over 100 elements

// Step 2: Compute sample standard deviation
FOR i := 1 TO 100 DO
    diff := REAL(InputArray[i]) - Mean;
    sumSqDiff := sumSqDiff + (diff * diff);
END_FOR;

// Use N-1 (99) for sample standard deviation
IF 99.0 > 0.0 THEN
    StdDev := SQRT(sumSqDiff / 99.0);
ELSE
    StdDev := 0.0; // Prevent division by zero if all values are identical
END_IF;

PROGRAM PLC_PRG
VAR
    StatCalculator: FB_MeanStdDevCalculator;

    RawSensorData : ARRAY[1..100] OF INT := [100, 102, 105, ...]; // Populate with actual data

    CurrentMean : REAL := 0.0;
    CurrentStdDev : REAL := 0.0;
END_VAR

// Run statistical computation
StatCalculator(
    InputArray := RawSensorData
);

CurrentMean := StatCalculator.Mean;
CurrentStdDev := StatCalculator.StdDev;
