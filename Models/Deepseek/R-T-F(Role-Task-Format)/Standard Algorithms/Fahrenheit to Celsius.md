FUNCTION_BLOCK FB_FahrenheitToCelsius
VAR_INPUT
    Fahrenheit : REAL; // Input temperature in Fahrenheit
END_VAR

VAR_OUTPUT
    Celsius : REAL := 0.0; // Output temperature in Celsius
    ValidInput : BOOL := FALSE; // Flag indicating if the input is valid
END_VAR

// Method to perform the conversion
METHOD Convert(this : REFERENCE TO FB_FahrenheitToCelsius)
BEGIN
    // Reset output variables
    this.Celsius := 0.0;
    this.ValidInput := FALSE;

    // Validate the input to ensure it is above absolute zero (-459.67°F)
    IF this.Fahrenheit >= -459.67 THEN
        // Perform the conversion from Fahrenheit to Celsius
        this.Celsius := (this.Fahrenheit - 32) * 5 / 9;
        this.ValidInput := TRUE;
    ELSE
        // Set fallback values for invalid input
        this.Celsius := 0.0;
        this.ValidInput := FALSE;
    END_IF;
END_METHOD

// Main execution method
METHOD MAIN(this : REFERENCE TO FB_FahrenheitToCelsius) : BOOL
BEGIN
    this.Convert();
    RETURN TRUE;
END_METHOD

PROGRAM MainProgram
VAR
    TempConverterFB : FB_FahrenheitToCelsius;
    FahrenheitTemp : REAL := 212.0; // Example Fahrenheit temperature
    CelsiusTemp : REAL;
    IsValid : BOOL;
END_VAR

// Set the Fahrenheit temperature to convert
TempConverterFB.Fahrenheit := FahrenheitTemp;

// Call the conversion function block
TempConverterFB();

// Get the converted Celsius temperature and validity flag
CelsiusTemp := TempConverterFB.Celsius;
IsValid := TempConverterFB.ValidInput;

// Output the results
IF IsValid THEN
    // Display or log the valid Celsius temperature
    // Display(CelsiusTemp); // Hypothetical display function
ELSE
    // Handle invalid input
    // Log("Invalid Fahrenheit temperature"); // Hypothetical logging function
END_IF;
