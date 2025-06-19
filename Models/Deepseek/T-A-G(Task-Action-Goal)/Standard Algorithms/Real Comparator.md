FUNCTION_BLOCK RealNumberComparatorFunctionBlock
VAR_INPUT
    Input1 : REAL;          // First input value
    Input2 : REAL;          // Second input value
    Precision : INT;        // Number of decimal places for comparison
    Enable : BOOL;          // Trigger execution
END_VAR

VAR_OUTPUT
    Equal : BOOL;           // Output flag if inputs are equal
    Greater : BOOL;         // Output flag if Input1 is greater than Input2
    Less : BOOL;            // Output flag if Input1 is less than Input2
    Error : BOOL;           // Output flag if there is an error (Precision < 0 or Enable = FALSE)
END_VAR

VAR
    Scale : REAL;            // Scaling factor based on Precision
    ScaledInput1 : INT;     // Scaled and rounded Input1
    ScaledInput2 : INT;     // Scaled and rounded Input2
END_VAR

METHOD Execute;
BEGIN
    // Initialize output flags
    Equal := FALSE;
    Greater := FALSE;
    Less := FALSE;
    Error := FALSE;

    // Check if Enable is TRUE and Precision is non-negative
    IF Enable AND Precision >= 0 THEN
        // Calculate the scaling factor
        Scale := POWER(10, Precision);

        // Scale and round the inputs
        ScaledInput1 := ROUND(Input1 * Scale);
        ScaledInput2 := ROUND(Input2 * Scale);

        // Compare the scaled and rounded values
        IF ScaledInput1 = ScaledInput2 THEN
            Equal := TRUE;
        ELSIF ScaledInput1 > ScaledInput2 THEN
            Greater := TRUE;
        ELSE
            Less := TRUE;
        END_IF;
    ELSE
        // Set error flag if Enable is FALSE or Precision is negative
        Error := TRUE;
    END_IF;
END_METHOD



