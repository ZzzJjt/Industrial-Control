FUNCTION_BLOCK LinearInterpolation
VAR_INPUT
    X1 : REAL; // X-coordinate of the first known point
    Y1 : REAL; // Y-coordinate of the first known point
    X2 : REAL; // X-coordinate of the second known point
    Y2 : REAL; // Y-coordinate of the second known point
    X  : REAL; // The value at which interpolation should occur
END_VAR

VAR_OUTPUT
    Y : REAL; // The interpolated output value
    Error : BOOL; // Indicates if an error occurred
    ErrorMessage : STRING; // Detailed error message
END_VAR

METHOD Execute : BOOL
BEGIN
    // Initialize outputs
    Y := Y1; // Default to Y1 in case of error or special cases
    Error := FALSE;
    ErrorMessage := '';

    // Check for division by zero
    IF X2 <> X1 THEN
        // Perform linear interpolation using the formula:
        // Y := Y1 + ((X - X1) * (Y2 - Y1)) / (X2 - X1)
        Y := Y1 + ((X - X1) * (Y2 - Y1)) / (X2 - X1);
    ELSE
        // Handle the case where X2 equals X1 to avoid division by zero
        Error := TRUE;
        ErrorMessage := 'Division by zero encountered: X2 = X1';
        // Optionally, set Y to a fallback value or handle as needed
        Y := Y1; // Fallback to Y1 if X2 = X1
    END_IF;

    RETURN TRUE;
END_METHOD

// Main execution loop
Execute();



