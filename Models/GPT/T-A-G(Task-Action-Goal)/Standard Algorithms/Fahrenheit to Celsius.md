FUNCTION_BLOCK FB_FahrenheitToCelsius
VAR_INPUT
    Fahrenheit : REAL;  // Input temperature in Fahrenheit
END_VAR

VAR_OUTPUT
    Celsius    : REAL;  // Output temperature in Celsius
    ValidInput : BOOL;  // TRUE if input is above absolute zero
END_VAR

VAR CONSTANT
    ABSOLUTE_ZERO_F : REAL := -459.67;  // Physical limit in Fahrenheit
END_VAR

// Input validation: Reject temperatures below absolute zero
IF Fahrenheit >= ABSOLUTE_ZERO_F THEN
    ValidInput := TRUE;

    // Conversion formula: C = (F - 32) * 5 / 9
    Celsius := (Fahrenheit - 32.0) * 5.0 / 9.0;

ELSE
    ValidInput := FALSE;

    // Output invalid marker value
    Celsius := 0.0; // Optional: Use -999.9 or another sentinel if preferred
END_IF;
