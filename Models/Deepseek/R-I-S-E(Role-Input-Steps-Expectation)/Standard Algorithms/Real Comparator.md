FUNCTION_BLOCK PrecisionComparatorFB
VAR_INPUT
    Input1 : REAL;       // First input value to compare
    Input2 : REAL;       // Second input value to compare
    Precision : INT;     // Number of decimal places for comparison
    Enable : BOOL;       // Enable flag to activate the comparator
END_VAR

VAR_OUTPUT
    Equal : BOOL;        // Flag indicating if the inputs are equal
    Greater : BOOL;      // Flag indicating if Input1 is greater than Input2
    Less : BOOL;         // Flag indicating if Input1 is less than Input2
    Error : BOOL;        // Flag indicating an error condition
END_VAR

VAR
    Scale : REAL;         // Scaling factor based on Precision
    Rounded1 : INT;       // Scaled and rounded Input1
    Rounded2 : INT;       // Scaled and rounded Input2
END_VAR

METHOD Execute : VOID
BEGIN
    // Initialize output flags
    Equal := FALSE;
    Greater := FALSE;
    Less := FALSE;
    Error := FALSE;

    // Validate inputs
    IF Precision < 0 OR NOT Enable THEN
        Error := TRUE;
        RETURN;
    END_IF;

    // Compute scaling factor
    Scale := POWER(10, Precision);

    // Scale and round inputs
    Rounded1 := REAL_TO_INT(Input1 * Scale);
    Rounded2 := REAL_TO_INT(Input2 * Scale);

    // Compare results
    IF Rounded1 = Rounded2 THEN
        Equal := TRUE;
    ELSIF Rounded1 > Rounded2 THEN
        Greater := TRUE;
    ELSIF Rounded1 < Rounded2 THEN
        Less := TRUE;
    ELSE
        Error := TRUE; // Fallback error condition
    END_IF;
END_METHOD

END_FUNCTION_BLOCK

PROGRAM MainProgram
VAR
    PCFB : PrecisionComparatorFB;
    Input1 : REAL := 123.456; // First input value
    Input2 : REAL := 123.457; // Second input value
    Precision : INT := 3;      // Comparison precision (3 decimal places)
    Enable : BOOL := TRUE;     // Enable flag
    Equal : BOOL;
    Greater : BOOL;
    Less : BOOL;
    Error : BOOL;
END_VAR

// Configure the function block with sample inputs
PCFB(Input1 := Input1,
     Input2 := Input2,
     Precision := Precision,
     Enable := Enable);

// Call the function block
PCFB.Execute();

// Use the outputs
Equal := PCFB.Equal;
Greater := PCFB.Greater;
Less := PCFB.Less;
Error := PCFB.Error;

// Example usage: Print or log the results
// Note: In a real PLC environment, you would typically use outputs to control other parts of the system
IF Error THEN
    // Handle error condition
ELSE
    IF Equal THEN
        // Inputs are equal
    ELSIF Greater THEN
        // Input1 is greater than Input2
    ELSIF Less THEN
        // Input1 is less than Input2
    END_IF;
END_IF;
END_PROGRAM
