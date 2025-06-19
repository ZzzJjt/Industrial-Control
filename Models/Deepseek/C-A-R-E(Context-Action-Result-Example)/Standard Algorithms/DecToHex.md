Certainly! Below is a self-contained function block in IEC 61131-3 Structured Text that converts a 10-digit decimal input (using the `LINT` type) to a hexadecimal string. The function block handles edge cases such as zero values and ensures accurate conversion without leading zeros.

### Decimal to Hexadecimal Conversion Function Block

```st
[<title="DecimalToHexConverter">]
FUNCTION_BLOCK DecimalToHexFB
VAR_INPUT
    Enable : BOOL;          // Enable signal to start the conversion
    DecValue : LINT;         // Decimal value to convert (up to 10 digits)
END_VAR

VAR_OUTPUT
    HexString : STRING[20]; // Hexadecimal string output
    Success : BOOL := FALSE; // Flag indicating successful conversion
END_VAR

VAR
    Remainder : INT;
    HexDigit : CHAR;
    DigitMap : ARRAY[0..15] OF CHAR := ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'A', 'B', 'C', 'D', 'E', 'F'];
    TempHexString : STRING[20];
    Index : INT;
BEGIN
    IF Enable THEN
        TempHexString := '';
        Index := 1;

        // Handle zero case
        IF DecValue = 0 THEN
            TempHexString := '0';
        ELSE
            WHILE DecValue > 0 DO
                Remainder := Modulo(DecValue, 16);
                HexDigit := DigitMap[Remainder];
                TempHexString[Index] := HexDigit;
                Index := Index + 1;
                DecValue := Div(DecValue, 16);
            END_WHILE;
        END_IF;

        // Reverse the string to get the correct order
        FOR Index := 1 TO LENGTH(TempHexString) DO
            HexString[Index] := TempHexString[LENGTH(TempHexString) - Index + 1];
        END_FOR;

        // Trim trailing null characters
        WHILE LENGTH(HexString) > 0 AND HexString[LENGTH(HexString)] = #0 THEN
            HexString := LEFT(HexString, LENGTH(HexString) - 1);
        END_WHILE;

        Success := TRUE;
    ELSE
        Success := FALSE;
    END_IF;
END_FUNCTION_BLOCK
```

### Explanation of the Code

#### Inputs
- **Enable**: A boolean flag to initiate the conversion.
- **DecValue**: A 10-digit decimal value to be converted to hexadecimal (using the `LINT` type).

#### Outputs
- **HexString**: The resulting hexadecimal string.
- **Success**: A boolean flag indicating whether the conversion was successful.

#### Internal Variables
- **Remainder**: Stores the remainder when dividing the decimal value by 16.
- **HexDigit**: Stores the corresponding hexadecimal character for the remainder.
- **DigitMap**: An array mapping remainders (0-15) to their respective hexadecimal characters ('0'-'F').
- **TempHexString**: Temporary storage for building the hexadecimal string in reverse order.
- **Index**: Used for indexing during string manipulation.

#### Logic
- **Zero Case Handling**: If `DecValue` is zero, directly set `TempHexString` to `'0'`.
- **Conversion Loop**:
  - While `DecValue` is greater than zero, calculate the remainder when divided by 16.
  - Map the remainder to its corresponding hexadecimal character using `DigitMap`.
  - Append the character to `TempHexString`.
  - Update `DecValue` by integer division by 16.
- **Reversing the String**: After the loop, reverse `TempHexString` to obtain the correct hexadecimal string order.
- **Trimming Null Characters**: Remove any trailing null characters from `HexString`.

### Usage Example

To use this function block in a PLC program, you would instantiate it and call its method:

```st
PROGRAM MainProgram
VAR
    MyConverter : DecimalToHexFB;
    DecValue : LINT := 1234567890;
    HexResult : STRING[20];
    ConversionSuccess : BOOL;
BEGIN
    // Configure the converter
    MyConverter.Enable := TRUE;
    MyConverter.DecValue := DecValue;

    // Execute the conversion
    MyConverter();

    // Retrieve the results
    HexResult := MyConverter.HexString;
    ConversionSuccess := MyConverter.Success;

    // Output the result
    IF ConversionSuccess THEN
        // Conversion was successful
        // Use HexResult as needed
    ELSE
        // Conversion failed
    END_IF;
END_PROGRAM
```

This function block is modular, reliable, and reusable, making it suitable for various applications in industrial automation systems where decimal-to-hexadecimal conversion is required. It ensures accuracy, supports a wide input range, and is easy to integrate into existing PLC programs.
