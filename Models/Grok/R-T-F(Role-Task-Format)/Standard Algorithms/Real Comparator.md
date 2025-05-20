(* IEC 61131-3 Structured Text function block for precision-aware real number comparison *)
(* Compares two REAL inputs with respect to a configurable number of decimal places *)
(* Uses scaling and integer comparison to handle floating-point rounding issues *)
(* Optimized for scan-cycle compatibility and industrial applications *)

FUNCTION_BLOCK RealComparator
VAR_INPUT
    Enable : BOOL; (* TRUE to enable comparison *)
    Input1 : REAL; (* First REAL number to compare *)
    Input2 : REAL; (* Second REAL number to compare *)
    Precision : INT := 2; (* Number of decimal places for comparison *)
END_VAR

VAR_OUTPUT
    Equal : BOOL; (* TRUE if inputs are equal within precision *)
    Greater : BOOL; (* TRUE if Input1 > Input2 within precision *)
    Less : BOOL; (* TRUE if Input1 < Input2 within precision *)
    Error : BOOL := FALSE; (* TRUE if invalid precision or disabled *)
END_VAR

VAR
    Scale : REAL; (* Scaling factor: 10^Precision *)
    Scaled1 : LREAL; (* Scaled and rounded Input1 *)
    Scaled2 : LREAL; (* Scaled and rounded Input2 *)
    MaxPrecision : INT := 6; (* Maximum precision to prevent overflow *)
END_VAR

(* Reset outputs when disabled *)
IF NOT Enable THEN
    Equal := FALSE;
    Greater := FALSE;
    Less := FALSE;
    Error := TRUE; (* Indicate disabled state *)
    RETURN;
END_IF;

(* Input validation *)
IF Precision < 0 OR Precision > MaxPrecision THEN
    Equal := FALSE;
    Greater := FALSE;
    Less := FALSE;
    Error := TRUE; (* Invalid precision *)
    RETURN;
END_IF;

(* Initialize outputs *)
Equal := FALSE;
Greater := FALSE;
Less := FALSE;
Error := FALSE;

(* Calculate scale factor: 10^Precision *)
Scale := POWER(10.0, TO_REAL(Precision));

(* Scale and round inputs to integers *)
Scaled1 := ROUND(Input1 * Scale);
Scaled2 := ROUND(Input2 * Scale);

(* Perform comparison *)
IF Scaled1 = Scaled2 THEN
    Equal := TRUE;
ELSIF Scaled1 > Scaled2 THEN
    Greater := TRUE;
ELSE
    Less := TRUE;
END_IF;

(* Execution Notes *)
(* - Scales inputs by 10^Precision and rounds to integers to avoid floating-point rounding issues. *)
(* - Uses LREAL for scaled values to handle large numbers at high precision. *)
(* - Limits Precision to 6 to prevent overflow (e.g., 10^6 fits within LREAL range). *)
(* - Executes in a single scan cycle (<1 ms), suitable for 10-50 ms PLC scan times. *)
(* - Ideal for alarm evaluation, setpoint matching, or process monitoring in industrial systems. *)
END_FUNCTION_BLOCK
