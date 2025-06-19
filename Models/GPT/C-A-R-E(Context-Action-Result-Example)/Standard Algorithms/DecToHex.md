FUNCTION_BLOCK FB_DecToHexConverter
VAR_INPUT
    Enable      : BOOL;     // Starts conversion when TRUE
    DecValue    : LINT;     // Input decimal value (up to 10 digits)
END_VAR

VAR_OUTPUT
    HexString   : STRING[16]; // Output hexadecimal string
    Done        : BOOL;       // Indicates completion of conversion
END_VAR

VAR
    TempValue   : ULINT;      // Use unsigned for division/modulo
    Remainder   : INT;
    DigitMap    : ARRAY[0..15] OF STRING[1] := ['0','1','2','3','4','5','6','7','8','9','A','B','C','D','E','F'];
    HexTemp     : STRING[16];
    Char        : STRING[1];
END_VAR

// Main conversion logic
IF Enable THEN
    HexTemp := '';                  // Reset temporary string
    TempValue := ULINT(DecValue);  // Convert signed to unsigned
    Done := FALSE;

    IF TempValue = 0 THEN
        HexTemp := '0';            // Edge case: zero
    ELSE
        WHILE TempValue > 0 DO
            Remainder := INT(TempValue MOD 16);
            Char := DigitMap[Remainder];
            HexTemp := CONCAT(Char, HexTemp); // Prepend new digit
            TempValue := TempValue / 16;
        END_WHILE;
    END_IF;

    HexString := HexTemp;
    Done := TRUE;
END_IF;
