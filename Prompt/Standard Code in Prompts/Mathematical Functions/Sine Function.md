```
FUNCTION_BLOCK SineCalculation
VAR_INPUT
    Angle : REAL; // Angle in radians for which sine is to be calculated
VAR_OUTPUT
    SineValue : REAL; // Computed sine value
VAR
    SeriesTerms : ARRAY [0..5] OF REAL; // Terms of the Taylor series expansion
    FactorialCache : ARRAY [0..5] OF REAL; // Cache for factorial calculations
    CurrentTerm : REAL; // Current term in the series expansion
    Sign : REAL := 1; // Sign of the current term (+1/-1)
END_VAR

// Initialize the factorial cache for the first few terms
FOR i := 0 TO 5 DO
    IF i = 0 THEN
        FactorialCache[i] := 1;
    ELSE
        FactorialCache[i] := FactorialCache[i-1] * REAL(i);
    END_IF;
END_FOR;

// Compute the sine using the Taylor series expansion
FOR i := 0 TO 5 DO
    CurrentTerm := POWER(Angle, 2*i + 1) / FactorialCache[2*i + 1];
    SeriesTerms[i] := CurrentTerm * Sign;
    Sign := -Sign; // Alternate the sign for the next term
END_FOR;

// Sum the series terms to get the sine value
FOR i := 0 TO 5 DO
    SineValue := SineValue + SeriesTerms[i];
END_FOR;

END_FUNCTION_BLOCK
```
