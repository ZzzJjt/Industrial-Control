FUNCTION_BLOCK SineFunction
VAR_INPUT
    AngleRad : REAL; // Input angle in radians
END_VAR

VAR_OUTPUT
    SineValue : REAL; // Computed sine of the input angle
END_VAR

METHOD Execute : BOOL
BEGIN
    // Use the built-in SIN() function to compute the sine of the input angle
    SineValue := SIN(AngleRad);

    RETURN TRUE;
END_METHOD

// Main execution loop
Execute();



