(* IEC 61131-3 Structured Text Function Block for Real Number Comparison *)
(* Compares two REAL numbers with user-defined precision in decimal places *)
(* Conforms to industrial automation best practices for PLC applications *)

FUNCTION_BLOCK RealComparator
VAR_INPUT
    Input1: REAL;          (* First number to compare, e.g., 123.456 *)
    Input2: REAL;          (* Second number to compare, e.g., 123.457 *)
    Precision: INT;        (* Number of decimal places for comparison, e.g., 2 *)
    Enable: BOOL;          (* TRUE to trigger comparison *)
END_VAR

VAR_OUTPUT
    Equal: BOOL;           (* TRUE if Input1 equals Input2 within precision *)
    Greater: BOOL;         (* TRUE if Input1 is greater than Input2 *)
    Less: BOOL;            (* TRUE if Input1 is less than Input2 *)
    Error: BOOL;           (* TRUE if Precision < 0 or Enable = FALSE *)
END_VAR

VAR
    Scale: REAL;           (* Scaling factor: 10^Precision *)
    Scaled1: DINT;         (* Scaled and rounded Input1 as integer *)
    Scaled2: DINT;         (* Scaled and rounded Input2 as integer *)
    ValidInput: BOOL;      (* TRUE if inputs are valid *)
    Temp: REAL;            (* Temporary variable for calculations *)
    i: INT;                (* Loop index for Scale calculation *)
END_VAR

(* Step 1: Input Validation *)
(* Check Enable and Precision, ensure inputs are reasonable *)
ValidInput := Enable AND (Precision >= 0) AND (Precision <= 6) 
              AND (ABS(Input1) < 1.0E6) AND (ABS(Input2) < 1.0E6);

(* Step 2: Handle Invalid Input or Disabled State *)
(* Reset outputs if not enabled or inputs are invalid *)
IF NOT ValidInput THEN
    Equal := FALSE;
    Greater := FALSE;
    Less := FALSE;
    Error := TRUE;
    RETURN;
END_IF;

(* Step 3: Calculate Scaling Factor *)
(* Scale = 10^Precision, e.g., Precision=2 -> Scale=100 *)
Scale := 1.0;
FOR i := 1 TO Precision DO
    Scale := Scale * 10.0;
END_FOR;

(* Step 4: Scale and Round Inputs *)
(* Multiply by Scale, round to nearest integer, convert to DINT *)
Temp := Input1 * Scale;
IF Temp >= 0.0 THEN
    Scaled1 := DINT_OF_REAL(Temp + 0.5);  (* Round up for positive *)
ELSE
    Scaled1 := DINT_OF_REAL(Temp - 0.5);  (* Round down for negative *)
END_IF;

Temp := Input2 * Scale;
IF Temp >= 0.0 THEN
    Scaled2 := DINT_OF_REAL(Temp + 0.5);
ELSE
    Scaled2 := DINT_OF_REAL(Temp - 0.5);
END_IF;

(* Step 5: Compare Scaled Values *)
(* Compare integers to determine relationship *)
Equal := (Scaled1 = Scaled2);
Greater := (Scaled1 > Scaled2);
Less := (Scaled1 < Scaled2);

(* Step 6: Clear Error Flag *)
(* No errors if comparison completes successfully *)
Error := FALSE;

(* Step 7: Safety Check *)
(* Ensure no numerical overflow or invalid states *)
IF ABS(Scaled1) > 2147483647 OR ABS(Scaled2) > 2147483647 THEN
    Equal := FALSE;
    Greater := FALSE;
    Less := FALSE;
    Error := TRUE;
END_IF;

END_FUNCTION_BLOCK
