(* IEC 61131-3 Structured Text Function Block for Decimal to Hexadecimal Conversion *)
(* Converts a 10-digit decimal LINT to a hexadecimal STRING *)
(* Conforms to industrial automation best practices for PLC applications *)

FUNCTION_BLOCK DecToHex
VAR_INPUT
    DecValue: LINT;                    (* Decimal input value, 0 to 9,999,999,999 *)
END_VAR

VAR_OUTPUT
    HexString: STRING[8];              (* Hexadecimal output string, max 8 chars *)
END_VAR

VAR
    TempValue: LINT;                   (* Temporary value for division *)
    Remainder: LINT;                   (* Remainder from division by 16 *)
    HexDigits: ARRAY[0..15] OF STRING[1] := ['0','1','2','3','4','5','6','7','8','9','A','B','C','D','E','F'];
                                       (* Mapping of remainders 0-15 to hex digits *)
    Buffer: ARRAY[1..8] OF STRING[1];  (* Buffer to store hex digits *)
    DigitCount: INT;                   (* Number of hex digits *)
    ValidInput: BOOL;                  (* Input validation flag *)
    i: INT;                            (* Loop index *)
END_VAR

(* Step 1: Input Validation *)
(* Ensure DecValue is within valid range: 0 to 9,999,999,999 *)
ValidInput := (DecValue >= 0) AND (DecValue <= 9999999999);

(* Step 2: Handle Invalid Input *)
(* Output empty string if input is invalid *)
IF NOT ValidInput THEN
    HexString := '';
    RETURN;
END_IF;

(* Step 3: Handle Special Case: DecValue = 0 *)
(* Output '0' for zero input *)
IF DecValue = 0 THEN
    HexString := '0';
    RETURN;
END_IF;

(* Step 4: Initialize Conversion *)
(* Clear buffer and set up temporary value *)
TempValue := DecValue;
DigitCount := 0;
FOR i := 1 TO 8 DO
    Buffer[i] := '';
END_FOR;

(* Step 5: Convert Decimal to Hexadecimal *)
(* Repeatedly divide by 16, collect remainders, map to hex digits *)
WHILE TempValue > 0 AND DigitCount < 8 DO
    (* Get remainder (0-15) *)
    Remainder := TempValue MOD 16;
    
    (* Map remainder to hex digit *)
    DigitCount := DigitCount + 1;
    Buffer[DigitCount] := HexDigits[Remainder];
    
    (* Update value for next iteration *)
    TempValue := TempValue / 16;
END_WHILE;

(* Step 6: Construct Hex String *)
(* Build string from buffer in reverse order *)
HexString := '';
FOR i := DigitCount DOWNTO 1 DO
    HexString := CONCAT(HexString, Buffer[i]);
END_FOR;

(* Step 7: Safety Check *)
(* Ensure output is not empty and within length *)
IF LEN(HexString) = 0 OR LEN(HexString) > 8 THEN
    HexString := '';
END_IF;

END_FUNCTION_BLOCK
