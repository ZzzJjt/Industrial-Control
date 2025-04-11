```
FUNCTION_BLOCK DecToHex
VAR_INPUT
    decimalValue : DINT; // Decimal input value
END_VAR
VAR_OUTPUT
    hexValue : STRING[9]; // Hexadecimal output string
END_VAR
VAR
    tempDecimal : DINT; // Temporary storage for the decimal value
    tempHex : STRING[9]; // Temporary storage for the hexadecimal string
    i : INT; // Loop index
END_VAR

// Initialize temporary variables
tempDecimal := decimalValue;
tempHex := '';

// Convert decimal to hexadecimal
FOR i := 1 TO 8 DO
    CASE tempDecimal MOD 16 OF
        0 TO 9 : tempHex := CHAR(#'0'+tempDecimal MOD 16) + tempHex;
        10 : tempHex := 'A' + tempHex;
        11 : tempHex := 'B' + tempHex;
        12 : tempHex := 'C' + tempHex;
        13 : tempHex := 'D' + tempHex;
        14 : tempHex := 'E' + tempHex;
        15 : tempHex := 'F' + tempHex;
    END_CASE;
    tempDecimal := tempDecimal DIV 16;
    IF tempDecimal = 0 THEN
        EXIT;
    END_IF;
END_FOR;

hexValue := tempHex;

RETURN;

END_FUNCTION_BLOCK

PROGRAM ExampleProgram
VAR
    decimalInput : DINT := 1234567890; // Example decimal value
    hexOutput : STRING[9]; // Variable to store the hexadecimal value
BEGIN
    // Call the DecToHex function block
    DecToHex(decimalValue:=decimalInput, hexValue:=hexOutput);

    // Output the hexadecimal value
    WRITE("Hexadecimal Value: ", hexOutput);
END_PROGRAM
```
