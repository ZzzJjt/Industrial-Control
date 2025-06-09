FUNCTION_BLOCK FB_DecimalToHex
VAR_INPUT
    DecValue : LINT;     // Input decimal value (0 .. 9999999999)
END_VAR

VAR_OUTPUT
    HexString : STRING[16];  // Output hexadecimal string
END_VAR

VAR
    Remainder   : INT;
    TempValue   : ULINT;             // Temporary value for safe division
    CharMap     : STRING := '0123456789ABCDEF';  // Hex digits
    TempString  : STRING[16] := '';  // Reversed buffer
    Char        : STRING[1];         // Single character holder
    i           : INT;               // Loop counter
END_VAR

// Safety check: Clamp input within LINT bounds
IF DecValue < 0 THEN
    HexString := '0';
ELSIF DecValue = 0 THEN
    HexString := '0';
ELSE
    TempValue := ULINT(DecValue);   // Promote to ULINT to avoid negative results

    // Loop: Convert to hex by division and mapping remainders
    WHILE TempValue > 0 DO
        Remainder := INT(TempValue MOD 16);
        Char := MID(CharMap, Remainder + 1, 1);  // Extract hex digit
        TempString := CONCAT(Char, TempString);  // Accumulate in correct order
        TempValue := TempValue / 16;
    END_WHILE;

    HexString := TempString;
END_IF
