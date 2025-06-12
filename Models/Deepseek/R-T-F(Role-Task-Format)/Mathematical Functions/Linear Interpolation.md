FUNCTION_BLOCK LinearInterpolation
VAR_INPUT
    X1 : REAL; // First reference point X coordinate
    Y1 : REAL; // First reference point Y coordinate
    X2 : REAL; // Second reference point X coordinate
    Y2 : REAL; // Second reference point Y coordinate
    X : REAL;  // Target input value for interpolation
END_VAR

VAR_OUTPUT
    Y : REAL; // Interpolated output value
    Valid : BOOL; // Flag indicating if the interpolation was valid
    Error : STRING[50]; // Error message in case of invalid computation
END_VAR

METHOD Execute : BOOL
BEGIN
    // Initialize outputs
    Y := 0.0;
    Valid := FALSE;
    Error := '';

    // Check if X2 equals X1 to prevent division by zero
    IF X2 <> X1 THEN
        // Perform linear interpolation using the formula:
        // Y := Y1 + ((X - X1) * (Y2 - Y1)) / (X2 - X1)
        Y := Y1 + ((X - X1) * (Y2 - Y1)) / (X2 - X1);
        Valid := TRUE;
    ELSE
        // Handle the case where X2 equals X1
        Y := Y1; // Default to Y1 to avoid undefined behavior
        Valid := FALSE;
        Error := 'Division by zero: X2 equals X1';
    END_IF;

    RETURN TRUE;
END_METHOD

END_FUNCTION_BLOCK
