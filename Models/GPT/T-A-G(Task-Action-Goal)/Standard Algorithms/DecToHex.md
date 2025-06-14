FUNCTION_BLOCK FB_DecToHex
VAR_INPUT
    DecValue : LINT;          // Input decimal number
END_VAR

VAR_OUTPUT
    HexString : STRING[16];   // Output hexadecimal string (max 16 hex digits for LINT)
END_VAR

VAR
    TempValue     : LINT;
    Remainder     : INT;
    HexChars      : STRING[16] := '0123456789ABCDEF';
    CharIndex     : INT;
    i             : INT;
    RevBuffer     : STRING[16];  // Temporary reverse string buffer
    Digit         : STRING[1];   // Temporary digit string
END_VAR

// Initialization
HexString := '';
RevBuffer := '';
TempValue := DecValue;

// Edge case: Input is zero
IF TempValue = 0 THEN
    HexString := '0';
ELSE
    // Convert decimal to hex (in reverse order)
    WHILE TempValue > 0 DO
        Remainder := INT(TempValue MOD 16);
        CharIndex := Remainder + 1; // 1-based indexing for STRING
        Digit := MID(HexChars, CharIndex, 1);
        RevBuffer := CONCAT(Digit, RevBuffer);
        TempValue := TempValue / 16;
    END_WHILE;

    // Set final result
    HexString := RevBuffer;
END_IF;
