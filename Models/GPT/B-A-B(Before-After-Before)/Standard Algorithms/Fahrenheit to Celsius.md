FUNCTION_BLOCK FB_FahrenheitToCelsius
VAR_INPUT
    Fahrenheit : REAL; // Temperature in °F
END_VAR

VAR_OUTPUT
    Celsius    : REAL; // Temperature in °C
    ValidInput : BOOL; // TRUE if input is valid (above absolute zero)
END_VAR

VAR CONSTANT
    ABSOLUTE_ZERO_F : REAL := -459.67; // Physical limit
END_VAR

// Validate input and perform conversion
IF Fahrenheit >= ABSOLUTE_ZERO_F THEN
    Celsius := (Fahrenheit - 32.0) * (5.0 / 9.0);
    ValidInput := TRUE;
ELSE
    Celsius := 0.0; // Default fallback
    ValidInput := FALSE;
END_IF
