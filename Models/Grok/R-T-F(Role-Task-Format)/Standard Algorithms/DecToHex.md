(* IEC 61131-3 Structured Text function block for decimal to hexadecimal conversion *)
(* Converts a 10-digit LINT decimal to a hexadecimal string *)
(* Optimized for modularity, efficiency, and PLC scan-cycle execution *)

FUNCTION_BLOCK DecimalToHex
VAR_INPUT
    Enable : BOOL; (* TRUE to enable conversion *)
    DecValue : LINT; (* Decimal input value to convert *)
END_VAR

VAR_OUTPUT
    HexString : STRING[16]; (* Hexadecimal output string, max 16 chars for 64-bit LINT *)
    Fault : BOOL := FALSE; (* TRUE if conversion error occurs *)
END_VAR

VAR
    TempValue : ULINT; (* Temporary unsigned value for conversion *)
    Digit : USINT; (* Current hexadecimal digit (0-15) *)
    TempString : STRING[16]; (* Temporary string for building result *)
    CharIndex : INT := 0; (* Index for building TempString *)
    FinalIndex : INT; (* Index for reversing string *)
    HexChar : STRING[1]; (* Single character for digit mapping *)
    IsNegative : BOOL; (* Flag for negative input *)
    i : INT; (* Loop variable for string reversal *)
END_VAR

(* Reset outputs when disabled *)
IF NOT Enable THEN
    HexString := '';
    Fault := FALSE;
    RETURN;
END_IF;

(* Initialize variables *)
Fault := FALSE;
TempString := '';
CharIndex := 0;
HexString := '';
IsNegative := (DecValue < 0);

(* Handle edge case: input = 0 *)
IF DecValue = 0 THEN
    HexString := '0';
    RETURN;
END_IF;

(* Convert to unsigned for consistent handling *)
IF IsNegative THEN
    TempValue := ULINT#9223372036854775808 + ULINT#ABS(DecValue); (* Handle negative LINT *)
ELSE
    TempValue := TO_ULINT(DecValue);
END_IF;

(* Perform decimal to hexadecimal conversion *)
WHILE TempValue > 0 AND CharIndex < 16 DO
    (* Extract least significant digit *)
    Digit := TO_USINT(TempValue MOD 16);
    
    (* Map digit to hexadecimal character *)
    CASE Digit OF
        0..9: HexChar := TO_STRING(Digit); (* 0-9 *)
        10: HexChar := 'A';
        11: HexChar := 'B';
        12: HexChar := 'C';
        13: HexChar := 'D';
        14: HexChar := 'E';
        15: HexChar := 'F';
    ELSE
        Fault := TRUE; (* Invalid digit *)
        RETURN;
    END_CASE;
    
    (* Append character to temporary string *)
    TempString := CONCAT(TempString, HexChar);
    CharIndex := CharIndex + 1;
    
    (* Divide by 16 to process next digit *)
    TempValue := TempValue / 16;
END_WHILE;

(* Check for string overflow *)
IF CharIndex > 16 THEN
    Fault := TRUE;
    HexString := '';
    RETURN;
END_IF;

(* Reverse string to get correct order *)
FinalIndex := CharIndex - 1;
HexString := '';
FOR i := FinalIndex DOWNTO 0 DO
    HexChar := MID(TempString, 1, i + 1); (* Extract single character *)
    HexString := CONCAT(HexString, HexChar);
END_FOR;

(* Trim leading zeros, but keep at least one digit *)
IF LEN(HexString) > 1 THEN
    WHILE LEFT(HexString, 1) = '0' AND LEN(HexString) > 1 DO
        HexString := RIGHT(HexString, LEN(HexString) - 1);
    END_WHILE;
END_IF;

(* Performance Notes *)
(* - Executes in a single scan cycle for typical inputs, with bounded iterations (max 16 for 64-bit LINT). *)
(* - Uses ULINT for negative number handling to avoid overflow issues. *)
(* - String operations are minimized to reduce CPU load in PLC environments. *)
(* - Fault handling ensures safe operation for invalid inputs or edge cases. *)
(* - Suitable for diagnostics, logging, or communication tasks in automation systems. *)
END_FUNCTION_BLOCK
