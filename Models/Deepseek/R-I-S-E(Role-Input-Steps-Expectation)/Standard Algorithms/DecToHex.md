FUNCTION_BLOCK DecimalToHexFB
VAR_INPUT
    DecValue : LINT;       // 10-digit decimal input value
END_VAR

VAR_OUTPUT
    HexString : STRING[20]; // Hexadecimal string output
END_VAR

VAR
    Remainder : INT;
    DigitMap : ARRAY[0..15] OF CHAR := ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'A', 'B', 'C', 'D', 'E', 'F'];
    TempString : STRING[20];
    Index : INT;
    MaxLINT : LINT := 9_223_372_036_854_775_807; // Maximum value for LINT
END_VAR

METHOD Execute : VOID
BEGIN
    // Initialize output and temporary variables
    HexString := '';
    TempString := '';
    Index := 1;

    // Check for invalid input
    IF DecValue < 0 OR DecValue > MaxLINT THEN
        HexString := 'ERROR';
        RETURN;
    END_IF;

    // Special case for zero
    IF DecValue = 0 THEN
        HexString := '0';
        RETURN;
    END_IF;

    // Convert decimal to hexadecimal
    WHILE DecValue > 0 DO
        Remainder := MOD(DecValue, 16);
        TempString[Index] := DigitMap[Remainder];
        Index := Index + 1;
        DecValue := DecValue DIV 16;
    END_WHILE;

    // Reverse the accumulated string to get the correct order
    FOR i := 1 TO Index - 1 DO
        HexString[i] := TempString[Index - i];
    END_FOR;

    // Null-terminate the string
    HexString[Index] := #0;
END_METHOD

END_FUNCTION_BLOCK

PROGRAM MainProgram
VAR
    DTHFB : DecimalToHexFB;
    HexResult : STRING[20];
END_VAR

// Configure the function block with a sample decimal value
DTHFB(DecValue := 123456789);

// Call the function block
DTHFB.Execute();

// Use the output
HexResult := DTHFB.HexString;

// Example usage: Print or log the result
// Note: In a real PLC environment, you would typically use outputs to control other parts of the system
IF HexResult <> '' AND HexResult <> 'ERROR' THEN
    // Process the hexadecimal result
ELSE
    // Handle error or invalid input
END_IF;
END_PROGRAM
