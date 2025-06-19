(* IEC 61131-3 Structured Text: Fahrenheit to Celsius Conversion Function Block *)
(* Purpose: Converts a Fahrenheit temperature to Celsius with input validation *)

FUNCTION_BLOCK FahrenheitToCelsius
VAR_INPUT
    Enable : BOOL;                  (* TRUE to initiate conversion *)
    Fahrenheit : REAL;               (* Input temperature in Fahrenheit *)
END_VAR
VAR_OUTPUT
    Celsius : REAL;                 (* Output temperature in Celsius *)
    ValidInput : BOOL;              (* TRUE if input is valid, FALSE otherwise *)
END_VAR
VAR
    AbsoluteZeroF : REAL := -459.67; (* Absolute zero in Fahrenheit *)
    ConversionFactor : REAL := 5.0 / 9.0; (* Conversion factor: 5/9 *)
END_VAR

(* Main Logic *)
IF NOT Enable THEN
    (* Reset outputs when disabled *)
    Celsius := 0.0;
    ValidInput := FALSE;
ELSE
    (* Input Validation *)
    IF Fahrenheit < AbsoluteZeroF THEN
        (* Input below absolute zero, invalid *)
        Celsius := 0.0;
        ValidInput := FALSE;
    ELSE
        (* Perform Conversion: C = (F - 32) * 5/9 *)
        Celsius := (Fahrenheit - 32.0) * ConversionFactor;
        ValidInput := TRUE;
    END_IF;
END_IF;

(* Notes:
   - Purpose: Converts a Fahrenheit temperature to Celsius for use in HVAC, thermal processing, or sensor integration.
   - Formula: C = (F - 32) * 5/9
   - Inputs:
     - Enable: Initiates conversion when TRUE.
     - Fahrenheit: REAL input temperature in Fahrenheit.
   - Outputs:
     - Celsius: REAL output temperature in Celsius.
     - ValidInput: BOOL, TRUE if input is valid (>= -459.67°F), FALSE otherwise.
   - Validation:
     - Checks if Fahrenheit >= -459.67°F (absolute zero).
     - Sets Celsius = 0.0 and ValidInput = FALSE for invalid inputs.
   - Safety:
     - Uses REAL for precise floating-point arithmetic.
     - Resets outputs when Enable = FALSE to ensure safe state.
   - Usage:
     - Reusable in PLC programs for temperature displays, logging, or control logic.
     - Suitable for scan-cycle-driven environments (e.g., CODESYS, TwinCAT).
   - Maintenance:
     - Clear comments and modular design aid debugging.
     - ValidInput flag supports diagnostic logging or HMI feedback.
   - Platform Notes:
     - Uses REAL for compatibility; LREAL can be used for higher precision if supported.
     - Assumes standard floating-point arithmetic support.
*)
END_FUNCTION_BLOCK
