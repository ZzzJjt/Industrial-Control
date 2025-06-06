FUNCTION_BLOCK FB_FahrenheitToCelsius
VAR_INPUT
    Enable      : BOOL;      // Trigger conversion when TRUE
    Fahrenheit  : REAL;      // Input temperature in °F
END_VAR

VAR_OUTPUT
    Celsius     : REAL;      // Output temperature in °C
    ValidInput  : BOOL;      // TRUE if input is valid
    Done        : BOOL;      // TRUE when conversion is completed
END_VAR

VAR CONSTANT
    ABS_ZERO_F  : REAL := -459.67; // Absolute zero in Fahrenheit
END_VAR

// Main logic
IF Enable THEN
    Done := FALSE;

    IF Fahrenheit >= ABS_ZERO_F THEN
        Celsius := (Fahrenheit - 32.0) * 5.0 / 9.0;
        ValidInput := TRUE;
    ELSE
        Celsius := 0.0;
        ValidInput := FALSE;
    END_IF;

    Done := TRUE;
END_IF;
