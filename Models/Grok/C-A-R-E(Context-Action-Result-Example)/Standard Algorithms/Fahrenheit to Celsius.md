FUNCTION_BLOCK FahrenheitToCelsiusFB
VAR_INPUT
    Enable: BOOL;           // Enable conversion
    Fahrenheit: REAL;       // Temperature input in Fahrenheit
END_VAR

VAR_OUTPUT
    Celsius: REAL;          // Temperature output in Celsius
    ValidInput: BOOL;       // TRUE if input is valid
END_VAR

VAR
    ABSOLUTE_ZERO_F: REAL := -459.67; // Absolute zero in Fahrenheit
END_VAR

// Reset outputs when disabled
IF NOT Enable THEN
    Celsius := 0.0;
    ValidInput := FALSE;
    RETURN;
END_IF;

// Input validation
IF Fahrenheit >= ABSOLUTE_ZERO_F THEN
    // Perform conversion: C = (F - 32) * 5/9
    Celsius := (Fahrenheit - 32.0) * 5.0 / 9.0;
    ValidInput := TRUE;
ELSE
    // Invalid input (below absolute zero)
    Celsius := 0.0;
    ValidInput := FALSE;
END_IF;
END_FUNCTION_BLOCK
