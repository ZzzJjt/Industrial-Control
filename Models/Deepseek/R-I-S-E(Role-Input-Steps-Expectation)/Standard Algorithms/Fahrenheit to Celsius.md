PROGRAM MainProgram
VAR
    FTCFB : FahrenheitToCelsiusFB;
    TempFahrenheit : REAL := 212.0; // Sample Fahrenheit value
    TempCelsius : REAL;
    IsValid : BOOL;
END_VAR

// Configure the function block with a sample Fahrenheit value
FTCFB(Fahrenheit := TempFahrenheit);

// Call the function block
FTCFB.Execute();

// Use the outputs
TempCelsius := FTCFB.Celsius;
IsValid := FTCFB.ValidInput;

// Example usage: Print or log the result
// Note: In a real PLC environment, you would typically use outputs to control other parts of the system
IF IsValid THEN
    // Process the valid Celsius temperature
ELSE
    // Handle invalid input
END_IF;
END_PROGRAM
