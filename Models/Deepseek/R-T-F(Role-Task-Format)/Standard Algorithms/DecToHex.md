FUNCTION_BLOCK FB_DecToHexConverter
VAR_INPUT
    DecValue : LINT; // Decimal input value to be converted
END_VAR

VAR_OUTPUT
    HexString : STRING[20]; // Output hexadecimal string
END_VAR

VAR
    TempValue : LINT; // Temporary variable to hold the current value during conversion
    Remainder : INT; // Remainder of division by 16
    CharMap : ARRAY[0..15] OF CHAR := ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'A', 'B', 'C', 'D', 'E', 'F']; // Character map for hex digits
    Index : INT := 0; // Index to build the hexadecimal string from the end
END_VAR

// Method to perform decimal to hexadecimal conversion
METHOD Convert(this : REFERENCE TO FB_DecToHexConverter)
BEGIN
    // Reset the output string
    this.HexString := '';

    // Handle the special case where the input value is 0
    IF this.DecValue = 0 THEN
        this.HexString := '0';
        RETURN;
    END_IF;

    // Copy the input value to a temporary variable
    this.TempValue := this.DecValue;

    // Convert the decimal value to hexadecimal
    WHILE this.TempValue > 0 DO
        // Calculate the remainder when divided by 16
        this.Remainder := MOD(INT(this.TempValue), 16);
        
        // Map the remainder to the corresponding hex character
        this.HexString[Index] := this.CharMap[this.Remainder];
        
        // Move to the next position in the string
        Index := Index + 1;
        
        // Divide the value by 16 for the next iteration
        this.TempValue := this.TempValue DIV 16;
    END_WHILE;

    // Reverse the string to get the correct order
    FOR i := 0 TO Index - 1 DO
        this.HexString[i] := this.HexString[Index - 1 - i];
    END_FOR;

    // Null-terminate the string
    this.HexString[Index] := #0;
END_METHOD

// Main execution method
METHOD MAIN(this : REFERENCE TO FB_DecToHexConverter) : BOOL
BEGIN
    this.Convert();
    RETURN TRUE;
END_METHOD
