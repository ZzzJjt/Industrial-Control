FUNCTION_BLOCK FahrenheitToCelsiusConverter
VAR_INPUT
    Fahrenheit : REAL; // Input temperature in Fahrenheit
END_VAR

VAR_OUTPUT
    Celsius : REAL;     // Output temperature in Celsius
    ValidInput : BOOL;  // Flag indicating if the input is valid
END_VAR

METHOD Execute;
BEGIN
    // Initialize outputs
    Celsius := 0.0;
    ValidInput := FALSE;

    // Input validation: Check if the input is above absolute zero (-459.67°F)
    IF Fahrenheit >= -459.67 THEN
        // Convert Fahrenheit to Celsius using the formula: Celsius = (Fahrenheit - 32) * 5/9
        Celsius := (Fahrenheit - 32) * 5 / 9;
        ValidInput := TRUE;
    ELSE
        // Invalid input: Set Celsius to 0.0 and ValidInput to FALSE
        Celsius := 0.0;
        ValidInput := FALSE;
    END_IF;
END_METHOD



