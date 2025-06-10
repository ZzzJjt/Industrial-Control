FUNCTION_BLOCK FB_FahrenheitToCelsius
VAR_INPUT
    Fahrenheit : REAL; // Input temperature in °F
END_VAR

VAR_OUTPUT
    Celsius    : REAL; // Converted temperature in °C
    ValidInput : BOOL; // TRUE if Fahrenheit is above absolute zero
END_VAR

VAR CONSTANT
    ABSOLUTE_ZERO_F : REAL := -459.67; // Minimum physically valid value
END_VAR

// === Input Validation and Conversion ===
IF Fahrenheit >= ABSOLUTE_ZERO_F THEN
    ValidInput := TRUE;
    Celsius := (Fahrenheit - 32.0) * (5.0 / 9.0); // Standard conversion formula
ELSE
    ValidInput := FALSE;
    Celsius := 0.0; // Safe fallback value when input is invalid
END_IF
