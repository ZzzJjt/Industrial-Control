```
FUNCTION_BLOCK MeanStdDev
VAR_INPUT
    Data : ARRAY [1..100] OF INT; // Input array of 100 integers
VAR_OUTPUT
    Mean : REAL; // Mean value
    StdDev : REAL; // Standard deviation
VAR
    Sum : REAL := 0; // Sum of the elements
    SumOfSquares : REAL := 0; // Sum of the squares of the elements
    N : INT := 100; // Number of elements in the array
    Element : REAL; // Temporary variable for element conversion
END_VAR

// Calculate the sum of the elements
FOR i := 1 TO N DO
    Element := REAL(Data[i]); // Convert integer to real to avoid overflow
    Sum := Sum + Element;
END_FOR

// Calculate the mean
Mean := Sum / REAL(N);

// Calculate the sum of the squares of the elements
FOR i := 1 TO N DO
    Element := REAL(Data[i]);
    SumOfSquares := SumOfSquares + (Element - Mean) * (Element - Mean);
END_FOR

// Calculate the standard deviation
StdDev := SQRT(SumOfSquares / REAL(N));

END_FUNCTION_BLOCK
```
