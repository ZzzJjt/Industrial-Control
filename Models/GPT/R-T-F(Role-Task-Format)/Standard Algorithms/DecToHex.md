FUNCTION_BLOCK FB_DecToHexConverter
VAR_INPUT
    DecValue   : LINT;           // 10-digit decimal input
END_VAR

VAR_OUTPUT
    HexString  : STRING[16];     // Hexadecimal output string
END_VAR

VAR
    TempValue      : ULINT;      // Temporary value for conversion
    Remainder      : ULINT;
    CharIndex      : INT;
    HexChar        : CHAR;
    HexChars       : STRING[16];
    Result         : STRING[16];
END_VAR

// === Character map for hexadecimal digits ===
HexChars := '0123456789ABCDEF';

// === Edge case: input is 0 ===
IF DecValue = 0 THEN
    HexString := '0';
ELSE
    TempValue := ULINT(DecValue); // Ensure unsigned division
    Result := '';

    // Convert by repeatedly dividing by 16 and mapping to hex chars
    WHILE TempValue > 0 DO
        Remainder := TempValue MOD 16;
        TempValue := TempValue / 16;
        CharIndex := INT(Remainder) + 1; // STRING index starts at 1
        HexChar := MID(HexChars, CharIndex, 1); // Get corresponding hex digit
        Result := CONCAT(HexChar, Result); // Build hex string from right to left
    END_WHILE

    HexString := Result;
END_IF
