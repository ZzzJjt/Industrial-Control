(* IEC 61131-3 Structured Text function block for Fahrenheit to Celsius conversion *)
(* Converts a Fahrenheit temperature to Celsius using: Celsius = (Fahrenheit - 32) * 5/9 *)
(* Validates input against absolute zero (-459.67°F) for physical correctness *)
(* Optimized for scan-cycle safety and industrial temperature control applications *)

FUNCTION_BLOCK FahrenheitToCelsius
VAR_INPUT
    Enable : BOOL; (* TRUE to enable conversion *)
    Fahrenheit : REAL; (* Temperature input in Fahrenheit *)
END_VAR

VAR_OUTPUT
    Celsius : REAL; (* Converted temperature in Celsius *)
    ValidInput : BOOL; (* TRUE if input is valid (>= -459.67°F) *)
END_VAR

VAR
    ABSOLUTE_ZERO_F : REAL := -459.67; (* Absolute zero in Fahrenheit *)
    CONVERSION_FACTOR : REAL := 5.0 / 9.0; (* Conversion factor: 5/9 *)
    OFFSET : REAL := 32.0; (* Fahrenheit offset for conversion *)
END_VAR

(* Reset outputs when disabled *)
IF NOT Enable THEN
    Celsius := 0.0;
    ValidInput := FALSE;
    RETURN;
END_IF;

(* Input validation *)
IF Fahrenheit < ABSOLUTE_ZERO_F THEN
    (* Reject temperatures below absolute zero *)
    Celsius := 0.0;
    ValidInput := FALSE;
ELSE
    (* Perform conversion: Celsius = (Fahrenheit - 32) * 5/9 *)
    Celsius := (Fahrenheit - OFFSET) * CONVERSION_FACTOR;
    ValidInput := TRUE;
END_IF;

(* Best Practices for Reuse *)
(* - Use REAL for high precision in temperature calculations, suitable for industrial sensors. *)
(* - Single-cycle execution ensures compatibility with PLC scan cycles (e.g., 10-50 ms). *)
(* - Validation against absolute zero prevents physically impossible inputs, enhancing safety. *)
(* - Modular design allows integration into larger control systems (e.g., PID loops, HMI displays). *)
(* - Default outputs (Celsius := 0.0, ValidInput := FALSE) provide safe fallback for invalid states. *)
END_FUNCTION_BLOCK
