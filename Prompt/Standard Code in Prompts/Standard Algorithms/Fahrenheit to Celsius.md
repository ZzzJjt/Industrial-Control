```
FUNCTION_BLOCK FahToCel
VAR_INPUT
    fahrenheit : REAL; // Temperature in Fahrenheit
END_VAR
VAR_OUTPUT
    celsius : REAL; // Temperature in Celsius
END_VAR

// Perform the conversion
celsius := (5.0 / 9.0) * (fahrenheit - 32.0);

RETURN;

END_FUNCTION_BLOCK

PROGRAM ExampleProgram
VAR
    fahrenheitInput : REAL := 32.0; // Example Fahrenheit temperature
    celsiusOutput : REAL; // Variable to store the Celsius temperature
BEGIN
    // Call the FahToCel function block
    FahToCel(fahrenheit:=fahrenheitInput, celsius:=celsiusOutput);

    // Output the Celsius temperature
    WRITE("Celsius Temperature: ", celsiusOutput);
END_PROGRAM

FUNCTION_BLOCK FahToCel
VAR_INPUT
    fahrenheit : REAL; // Temperature in Fahrenheit
END_VAR
VAR_OUTPUT
    celsius : REAL; // Temperature in Celsius
END_VAR

// Perform the conversion if the input is valid
IF fahrenheit BETWEEN -273.15 AND 1000 THEN // Reasonable range check
    celsius := (5.0 / 9.0) * (fahrenheit - 32.0);
ELSE
    celsius := INVALID_VALUE; // Placeholder for invalid value handling
END_IF;

RETURN;

END_FUNCTION_BLOCK
```
