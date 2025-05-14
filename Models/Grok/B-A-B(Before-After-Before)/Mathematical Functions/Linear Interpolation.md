(* Function Block: LinearInterpolation
   Purpose: Performs linear interpolation to estimate Y for a given X between two points (X1,Y1) and (X2,Y2).
   Features:
   - Inputs: X1, Y1 (first point), X2, Y2 (second point), X (target X-value)
   - Output: Y (interpolated Y-value)
   - Formula: Y = Y1 + ((X - X1) * (Y2 - Y1)) / (X2 - X1)
   - Handles edge cases: division by zero, numerical stability
   - Optimized for PLCs: minimal operations, robust error checking
   Notes:
   - Assumes 32-bit REAL arithmetic (IEEE 754 single-precision)
   - Checks for small denominators to prevent overflow
   - Suitable for HMI scaling, analog conversion, PID setpoint adjustments
*)

FUNCTION_BLOCK LinearInterpolation
VAR_INPUT
    X1 : REAL;      (* X-coordinate of first point *)
    Y1 : REAL;      (* Y-coordinate of first point *)
    X2 : REAL;      (* X-coordinate of second point *)
    Y2 : REAL;      (* Y-coordinate of second point *)
    X : REAL;       (* Target X-value for interpolation *)
END_VAR
VAR_OUTPUT
    Y : REAL;       (* Interpolated Y-value *)
    ErrorFlag : BOOL; (* TRUE if computation fails (e.g., division by zero) *)
END_VAR
VAR
    Denominator : REAL;      (* X2 - X1 for slope calculation *)
    Slope : REAL;           (* (Y2 - Y1) / (X2 - X1) *)
    DeltaX : REAL;          (* X - X1 *)
    MinDenominator : REAL := 0.000001; (* Threshold for numerical stability *)
END_VAR

(* Initialize outputs *)
Y := 0.0;
ErrorFlag := FALSE;

(* Calculate denominator *)
Denominator := X2 - X1;

(* Check for division by zero or numerical instability *)
IF ABS(Denominator) < MinDenominator THEN
    (* Edge case: X1 ≈ X2, return Y1 and flag error *)
    Y := Y1;
    ErrorFlag := TRUE;
ELSE
    (* Compute slope: (Y2 - Y1) / (X2 - X1) *)
    Slope := (Y2 - Y1) / Denominator;
    
    (* Compute DeltaX: X - X1 *)
    DeltaX := X - X1;
    
    (* Linear interpolation: Y = Y1 + Slope * DeltaX *)
    Y := Y1 + Slope * DeltaX;
    
    (* Check for numerical overflow *)
    IF ABS(Y) > 1.0E6 THEN
        Y := Y1; (* Fallback to Y1 *)
        ErrorFlag := TRUE;
    END_IF
END_IF

END_FUNCTION_BLOCK
