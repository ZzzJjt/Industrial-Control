FUNCTION_BLOCK DecimalToHexConverter
VAR_INPUT
    DecValue : LINT; // Input decimal value (10-digit)
END_VAR

VAR_OUTPUT
    HexString : STRING[20]; // Output hexadecimal string
END_VAR

VAR
    TempValue : LINT;       // Temporary variable to hold the current value during conversion
    Remainder : INT;        // Remainder of division by 16
    Index : INT := 1;       // Index for constructing the hex string
    DigitMap : ARRAY[0..15] OF CHAR := ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'A', 'B', 'C', 'D', 'E', 'F']; // Mapping of remainders to hex digits
END_VAR

METHOD Execute;
BEGIN
    // Initialize output string
    HexString := '';

    // Handle the special case where the input is zero
    IF DecValue = 0 THEN
        HexString := '0';
        RETURN;
    END_IF;

    // Copy the input value to a temporary variable
    TempValue := DecValue;

    // Loop to convert decimal to hexadecimal
    WHILE TempValue > 0 DO
        // Calculate remainder when divided by 16
        Remainder := MOD(TempValue, 16);
        
        // Map remainder to corresponding hex digit
        HexString[Index] := DigitMap[Remainder];
        
        // Move to the next character position
        Index := Index + 1;
        
        // Divide the value by 16 for the next iteration
        TempValue := TempValue DIV 16;
    END_WHILE;

    // Reverse the constructed hex string
    FOR i := 1 TO Index - 1 DO
        HexString[i] := HexString[Index - i];
    END_FOR;

    // Null-terminate the string
    HexString[Index] := #0;
END_METHOD



