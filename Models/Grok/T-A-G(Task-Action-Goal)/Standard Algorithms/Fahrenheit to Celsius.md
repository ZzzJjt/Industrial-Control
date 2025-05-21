(* IEC 61131-3 Structured Text Function Block for Fahrenheit to Celsius Conversion *)
(* Converts a Fahrenheit temperature to Celsius with input validation *)
(* Conforms to industrial automation best practices for PLC applications *)

FUNCTION_BLOCK FahrenheitToCelsius
VAR_INPUT
    Fahrenheit: REAL;                   (* Input temperature in Fahrenheit (°F) *)
END_VAR

VAR_OUTPUT
    Celsius: REAL;                     (* Converted temperature in Celsius (°C) *)
    ValidInput: BOOL;                  (* TRUE if input is valid, FALSE otherwise *)
END_VAR

VAR
    AbsoluteZeroF: REAL := -459.67;    (* Absolute zero in Fahrenheit (°F) *)
END_VAR

(* Step 1: Input Validation *)
(* Check if Fahrenheit is at or above absolute zero (-459.67°F) *)
ValidInput := (Fahrenheit >= AbsoluteZeroF);

(* Step 2: Perform Conversion *)
(* Apply formula: Celsius = (Fahrenheit - 32) * 5/9 *)
IF ValidInput THEN
    Celsius := (Fahrenheit - 32.0) * (5.0 / 9.0);
ELSE
    Celsius := 0.0;                    (* Default output for invalid input *)
END_IF;

(* Step 3: Safety Check *)
(* Ensure output is reasonable; redundant for formula but good practice *)
IF ValidInput AND (Celsius < -273.15) THEN
    ValidInput := FALSE;               (* Invalidate if output below absolute zero *)
    Celsius := 0.0;
END_IF;

END_FUNCTION_BLOCK
