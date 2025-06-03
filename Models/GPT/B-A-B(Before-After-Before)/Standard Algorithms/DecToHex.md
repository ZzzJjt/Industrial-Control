FUNCTION_BLOCK FB_DecToHex
VAR_INPUT
    DecValue : LINT; // Accepts values up to 9999999999
END_VAR

VAR_OUTPUT
    HexString : STRING[16]; // Output hex string (max 16 characters for LINT)
END_VAR

VAR
    TempValue     : ULINT;               // Temporary unsigned value
    Remainder     : USINT;               // 0..15
    HexChar       : STRING[1];           // One hex character
    HexBuffer     : STRING[16];          // Build string in reverse
    CharIndex     : INT := 0;
    FinalString   : STRING[16] := '';
    HexLookup     : ARRAY[0..15] OF STRING[1] := 
        ['0','1','2','3','4','5','6','7','8','9','A','B','C','D','E','F'];
END_VAR

// Initialize
TempValue := ULINT(DecValue);
HexBuffer := '';
CharIndex := 0;

// Handle zero explicitly
IF TempValue = 0 THEN
    HexString := '0';
ELSE
    WHILE TempValue > 0 DO
        Remainder := USINT(TempValue MOD 16);
        HexChar := HexLookup[Remainder];
        HexBuffer := CONCAT(HexChar, HexBuffer);
        TempValue := TempValue / 16;
    END_WHILE
    HexString := HexBuffer;
END_IF
