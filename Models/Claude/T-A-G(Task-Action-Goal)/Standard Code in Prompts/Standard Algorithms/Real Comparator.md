```
FUNCTION_BLOCK RealComparator
VAR_INPUT
    input1 : REAL; // First input value
    input2 : REAL; // Second input value
    precision : INT; // Number of decimal places to compare up to
END_VAR
VAR_OUTPUT
    isEqual : BOOL; // Indicates if the two inputs are equal up to the specified precision
END_VAR
VAR
    multiplier : REAL; // Multiplier for shifting the decimal point
    roundedInput1 : REAL; // Rounded version of the first input
    roundedInput2 : REAL; // Rounded version of the second input
END_VAR

// Calculate the multiplier to shift the decimal point
multiplier := POWER(10.0, precision);

// Round both inputs to the specified precision
roundedInput1 := ROUND(input1 * multiplier) / multiplier;
roundedInput2 := ROUND(input2 * multiplier) / multiplier;

// Compare the rounded inputs
isEqual := roundedInput1 = roundedInput2;

RETURN;

END_FUNCTION_BLOCK

PROGRAM ExampleProgram
VAR
    value1 : REAL := 123.456789; // First real value
    value2 : REAL := 123.456788; // Second real value
    precisionLevel : INT := 6; // Number of decimal places to compare
    comparisonResult : BOOL; // Result of the comparison
BEGIN
    // Call the RealComparator function block
    RealComparator(input1:=value1, input2:=value2, precision:=precisionLevel, isEqual:=comparisonResult);
    
    // Output the result
    WRITE('Are the values equal up to ', precisionLevel, ' decimal places? ', comparisonResult);
END_PROGRAM
```
