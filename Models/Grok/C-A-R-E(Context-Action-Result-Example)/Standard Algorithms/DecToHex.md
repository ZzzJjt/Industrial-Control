FUNCTION_BLOCK DecToHexFB
VAR_INPUT
    Enable: BOOL;           // Enable conversion
    DecValue: LINT;         // Decimal input (up to 10 digits)
END_VAR

VAR_OUTPUT
    HexString: STRING[16];   // Hexadecimal output string
    Error: BOOL;            // TRUE if conversion fails
END_VAR

VAR
    TempValue: LINT;        // Temporary value for processing
    Remainder: LINT;        // Remainder from division
    DigitMap: ARRAY[0..15] OF STRING[1] := ['0', '1', '2', '3', '4', '5', '6', '7', 
                                            '8', '9', 'A', 'B', 'C', 'D', 'E', 'F']; // Hex digit mapping
    TempString: STRING[16]; // Temporary string for building result
    i: INT;                 // Loop counter
    ValidInput: BOOL;       // Input validation flag
END_VAR

// Reset outputs on disable
IF NOT Enable THEN
    HexString := '';
    Error := FALSE;
    RETURN;
END_IF;

// Input validation
ValidInput := TRUE;
Error := FALSE;
TempString := '';
HexString := '';

// Handle negative numbers (LINT range: -2^63 to 2^63-1)
IF DecValue < 0 THEN
    ValidInput := FALSE;
    Error := TRUE;
    HexString := 'NEGATIVE';
    RETURN;
END_IF;

// Initialize temporary value
TempValue := DecValue;

// Handle zero case
IF TempValue = 0 THEN
    HexString := '0';
    RETURN;
END_IF;

// Conversion loop: divide by 16, map remainder, build string
i := 0;
WHILE TempValue > 0 AND i < 16 AND NOT Error DO
    Remainder := TempValue MOD 16;
    IF Remainder >= 0 AND Remainder <= 15 THEN
        // Prepend hex digit to temporary string
        TempString := CONCAT(DigitMap[Remainder], TempString);
        TempValue := TempValue / 16;
        i := i + 1;
    ELSE
        // Invalid remainder (should not occur)
        Error := TRUE;
        HexString := 'ERROR';
        RETURN;
    END_IF;
END_WHILE;

// Check for string length overflow
IF i >= 16 AND TempValue > 0 THEN
    Error := TRUE;
    HexString := 'OVERFLOW';
    RETURN;
END_IF;

// Finalize output
IF TempString <> '' THEN
    HexString := TempString;
ELSE
    // Fallback for unexpected empty string
    Error := TRUE;
    HexString := 'ERROR';
END_IF;
END_FUNCTION_BLOCK
