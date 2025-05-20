FUNCTION_BLOCK LinearInterpolation
VAR_INPUT
    X1 : REAL; (* X-coordinate of first reference point *)
    Y1 : REAL; (* Y-coordinate of first reference point *)
    X2 : REAL; (* X-coordinate of second reference point *)
    Y2 : REAL; (* Y-coordinate of second reference point *)
    X : REAL;  (* Target X value for interpolation *)
END_VAR
VAR_OUTPUT
    Y : REAL;  (* Interpolated Y value *)
    ErrorCode : INT := 0; (* 0: Success, 1: Invalid input, 2: Division by zero *)
END_VAR
VAR
    DeltaX : REAL; (* Difference X2 - X1 *)
    DeltaY : REAL; (* Difference Y2 - Y1 *)
    Slope : REAL;  (* Slope (DeltaY / DeltaX) *)
END_VAR

(* Initialize output *)
Y := 0.0;
ErrorCode := 0;

(* Validate inputs for non-finite values *)
IF NOT IS_VALID_REAL(X1) OR NOT IS_VALID_REAL(Y1) OR 
   NOT IS_VALID_REAL(X2) OR NOT IS_VALID_REAL(Y2) OR 
   NOT IS_VALID_REAL(X) THEN
    ErrorCode := 1; (* Invalid input *)
    Y := Y1; (* Fallback to Y1 *)
    RETURN;
END_IF;

(* Compute differences *)
DeltaX := X2 - X1;
DeltaY := Y2 - Y1;

(* Check for division by zero *)
IF ABS(DeltaX) < 1.0E-6 THEN
    (* If X1 ≈ X2, return Y1 as fallback to avoid division by zero *)
    ErrorCode := 2; (* Division by zero *)
    Y := Y1;
ELSE
    (* Compute slope: (Y2 - Y1) / (X2 - X1) *)
    Slope := DeltaY / DeltaX;
    
    (* Compute interpolated value: Y = Y1 + (X - X1) * Slope *)
    Y := Y1 + (X - X1) * Slope;
    
    (* Ensure output is finite *)
    IF NOT IS_VALID_REAL(Y) THEN
        ErrorCode := 1; (* Invalid result *)
        Y := Y1; (* Fallback to Y1 *)
    END_IF;
END_IF;

(* Helper function to check if a REAL value is valid *)
METHOD IS_VALID_REAL : BOOL
VAR_INPUT
    Value : REAL;
END_VAR
IS_VALID_REAL := NOT (Value = 0.0 / 0.0) AND NOT (Value = 1.0 / 0.0);
END_METHOD

END_FUNCTION_BLOCK
