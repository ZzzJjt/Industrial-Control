Certainly! Below is a self-contained function block in IEC 61131-3 Structured Text that compares two REAL inputs with a configurable number of decimal places. The function block rounds both inputs to the specified precision and sets output flags for equality, greater than, and less than comparisons. It also includes input validation and an error flag for invalid precision values.

### Configurable Real Comparator Function Block

```st
[<title="RealComparatorFunctionBlock">]
FUNCTION_BLOCK RealComparatorFB
VAR_INPUT
    Enable : BOOL;          // Enable signal to start comparison
    Input1 : REAL;           // First input value
    Input2 : REAL;           // Second input value
    Precision : INT;         // Number of decimal places for comparison
END_VAR

VAR_OUTPUT
    Equal : BOOL := FALSE;   // Output flag indicating if inputs are equal
    Greater : BOOL := FALSE; // Output flag indicating if Input1 is greater than Input2
    Less : BOOL := FALSE;    // Output flag indicating if Input1 is less than Input2
    Error : BOOL := FALSE;   // Output flag indicating an error (invalid precision)
END_VAR

VAR
    Scale : REAL;
    Rounded1 : INT;
    Rounded2 : INT;
BEGIN
METHOD ExecuteComparison : VOID
BEGIN
    IF Enable THEN
        // Validate precision
        IF Precision < 0 THEN
            Error := TRUE;
            Equal := FALSE;
            Greater := FALSE;
            Less := FALSE;
        ELSE
            Error := FALSE;

            // Calculate scale factor
            Scale := POWER(10, Precision);

            // Round inputs to the specified precision
            Rounded1 := REAL_TO_INT(Input1 * Scale);
            Rounded2 := REAL_TO_INT(Input2 * Scale);

            // Perform comparisons
            Equal := (Rounded1 = Rounded2);
            Greater := (Rounded1 > Rounded2);
            Less := (Rounded1 < Rounded2);
        END_IF;
    ELSE
        Error := FALSE;
        Equal := FALSE;
        Greater := FALSE;
        Less := FALSE;
    END_IF;
END_METHOD

// Main execution method
ExecuteComparison();
END_FUNCTION_BLOCK
```

### Explanation of the Code

#### Inputs
- **Enable**: A boolean signal to initiate the comparison process.
- **Input1**: The first real number to be compared.
- **Input2**: The second real number to be compared.
- **Precision**: The number of decimal places for comparison.

#### Outputs
- **Equal**: A boolean flag indicating if `Input1` is equal to `Input2`.
- **Greater**: A boolean flag indicating if `Input1` is greater than `Input2`.
- **Less**: A boolean flag indicating if `Input1` is less than `Input2`.
- **Error**: A boolean flag indicating an error (e.g., invalid precision).

#### Internal Variables
- **Scale**: The scaling factor based on the precision.
- **Rounded1**: The rounded value of `Input1`.
- **Rounded2**: The rounded value of `Input2`.

#### Method: ExecuteComparison
- **Validation**: Checks if the precision is valid (non-negative). If not, sets the `Error` flag and clears other flags.
- **Scaling**: Calculates the scaling factor using `POWER(10, Precision)`.
- **Rounding**: Rounds `Input1` and `Input2` to the specified precision using `REAL_TO_INT`.
- **Comparisons**: Performs equality, greater-than, and less-than comparisons on the rounded values and sets the respective flags.

### Usage Example

To use this function block in a PLC program, you would instantiate it and call its `ExecuteComparison` method:

```st
PROGRAM MainProgram
VAR
    MyComparator : RealComparatorFB;
    Input1 : REAL := 123.456; // Example first input
    Input2 : REAL := 123.457; // Example second input
    Precision : INT := 3;     // Example precision
    Equal : BOOL;
    Greater : BOOL;
    Less : BOOL;
    Error : BOOL;
BEGIN
    // Configure the comparator
    MyComparator.Enable := TRUE;
    MyComparator.Input1 := Input1;
    MyComparator.Input2 := Input2;
    MyComparator.Precision := Precision;

    // Execute the comparison
    MyComparator();

    // Retrieve the results
    Equal := MyComparator.Equal;
    Greater := MyComparator.Greater;
    Less := MyComparator.Less;
    Error := MyComparator.Error;

    // Output the results
    IF Error THEN
        // Handle error (invalid precision)
    ELSIF Equal THEN
        // Inputs are equal
    ELSIF Greater THEN
        // Input1 is greater than Input2
    ELSIF Less THEN
        // Input1 is less than Input2
    END_IF;
END_PROGRAM
```

### Key Features
- **Configurable Precision**: Allows specifying the number of decimal places for comparison.
- **Robust Validation**: Validates precision to prevent errors.
- **Scan-Cycle Safety**: Suitable for PLC environments with scan-cycle constraints.
- **Modular Design**: Clear inputs and outputs for easy integration into broader control programs.
- **Clarity in Logic**: Easy-to-understand logic for comparing floating-point numbers.

This function block ensures stable and reliable logic decisions by accurately comparing REAL values up to a specified number of decimal places, making it suitable for various applications in industrial automation where precise comparisons are required.
