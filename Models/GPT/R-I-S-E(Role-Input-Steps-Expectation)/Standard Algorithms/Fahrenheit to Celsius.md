FUNCTION_BLOCK FB_FahrenheitToCelsius
VAR_INPUT
    Fahrenheit : REAL;     // Input temperature in °F
END_VAR

VAR_OUTPUT
    Celsius     : REAL;    // Output temperature in °C
    ValidInput  : BOOL;    // TRUE if input >= absolute zero (−459.67°F)
END_VAR

VAR CONSTANT
    ABS_ZERO_F : REAL := -459.67; // Absolute zero in Fahrenheit
END_VAR

// -- Input Validation & Conversion --
IF Fahrenheit >= ABS_ZERO_F THEN
    // Valid input: perform conversion using Celsius = (F − 32) × 5/9
    Celsius := (Fahrenheit - 32.0) * 5.0 / 9.0;
    ValidInput := TRUE;
ELSE
    // Invalid input: below physical threshold
    Celsius := 0.0;
    ValidInput := FALSE;
END_IF;
