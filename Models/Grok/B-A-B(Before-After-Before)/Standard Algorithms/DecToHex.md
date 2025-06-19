(* IEC 61131-3 Structured Text: Decimal to Hexadecimal Conversion Function Block *)
(* Purpose: Converts a 10-digit decimal value (up to 9999999999) to a hexadecimal string *)

FUNCTION_BLOCK DecimalToHex
VAR_INPUT
    Enable : BOOL;                  (* TRUE to initiate conversion *)
    DecValue : LINT;                (* Decimal input value, 0 to 9999999999 *)
END_VAR
VAR_OUTPUT
    HexString : STRING[16];         (* Hexadecimal output string, max 10 digits *)
    Error : BOOL;                   (* TRUE if input validation fails *)
END_VAR
VAR
    TempValue : LINT;               (* Temporary value for division *)
    Remainder : LINT;               (* Remainder for hex digit *)
    HexDigits : ARRAY[0..9] OF BYTE; (* Temporary array for hex digits *)
    DigitCount : INT := 0;          (* Number of hex digits *)
    i : INT;                        (* Loop counter *)
    Valid : BOOL;                   (* Input validation flag *)
    Char : BYTE;                    (* Current hex character *)
END_VAR

(* Main Logic *)
IF NOT Enable THEN
    (* Reset outputs when disabled *)
    HexString := '';
    Error := FALSE;
    DigitCount := 0;
ELSE
    (* Input Validation *)
    IF DecValue < 0 OR DecValue > 9999999999 THEN
        (* Input out of range *)
        Error := TRUE;
        HexString := '';
        DigitCount := 0;
    ELSE
        Error := FALSE;
        DigitCount := 0;
        TempValue := DecValue;
        
        (* Convert Decimal to Hex *)
        IF TempValue = 0 THEN
            (* Special case: 0 *)
            HexDigits[0] := 48;  (* ASCII '0' *)
            DigitCount := 1;
        ELSE
            (* Iterative division by 16 *)
            WHILE TempValue > 0 AND DigitCount < 10 DO
                Remainder := TempValue MOD 16;
                IF Remainder < 10 THEN
                    (* Digits 0-9: ASCII 48-57 *)
                    HexDigits[DigitCount] := 48 + Remainder;
                ELSE
                    (* Letters A-F: ASCII 65-70 *)
                    HexDigits[DigitCount] := 55 + Remainder;  (* 55 = 'A' - 10 *)
                END_IF;
                TempValue := TempValue / 16;
                DigitCount := DigitCount + 1;
            END_WHILE;
        END_IF;
        
        (* Build Hex String *)
        HexString := '';
        FOR i := DigitCount - 1 TO 0 BY -1 DO
            (* Append digits in reverse order *)
            Char := HexDigits[i];
            IF LEN(HexString) < 16 THEN
                HexString := CONCAT(CHR(Char), HexString);
            ELSE
                (* String overflow, truncate *)
                Error := TRUE;
                EXIT;
            END_IF;
        END_FOR;
    END_IF;
END_IF;

(* Notes:
   - Purpose: Converts a 10-digit decimal value (0 to 9999999999) to a hexadecimal string.
   - Inputs:
     - Enable: Initiates conversion when TRUE.
     - DecValue: LINT input value (0 to 9999999999).
   - Outputs:
     - HexString: STRING containing hex representation (e.g., '2540BE3FF' for 9999999999).
     - Error: TRUE if DecValue is negative, exceeds 9999999999, or string overflows.
   - Logic:
     - Divides DecValue by 16 iteratively, storing remainders (0-15) as hex digits.
     - Converts remainders to ASCII: 0-9 (48-57), A-F (65-70).
     - Builds string in reverse order for correct representation.
   - Edge Cases:
     - Negative values: Returns Error = TRUE, empty HexString.
     - Values > 9999999999: Returns Error = TRUE.
     - Zero: Returns '0'.
     - String overflow: Truncates and sets Error = TRUE.
   - Safety:
     - Validates input range to prevent errors.
     - Limits DigitCount to 10 to avoid array overflow.
     - Uses LEN check to prevent string overflow.
   - Platform Notes:
     - Uses LINT for 10-digit support; replace with DINT for 32-bit platforms (max 2147483647).
     - Assumes STRING supports 16 characters; adjust for platform limits.
   - Usage:
     - Ideal for diagnostics, memory addressing, or protocol formatting in PLC systems.
     - Reusable in HMI displays, logging, or communication tasks.
   - Maintenance:
     - Modular design with clear comments aids debugging.
     - Error flag supports diagnostic logging.
*)
END_FUNCTION_BLOCK
