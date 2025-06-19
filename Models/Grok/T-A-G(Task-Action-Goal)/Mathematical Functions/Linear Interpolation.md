(* Function Block: Linear Interpolation between Two Points *)
FUNCTION_BLOCK FB_LinearInterpolation
VAR_INPUT
    Execute : BOOL;                   (* Trigger interpolation computation *)
    X1 : REAL;                        (* X-coordinate of first known point *)
    Y1 : REAL;                        (* Y-coordinate of first known point *)
    X2 : REAL;                        (* X-coordinate of second known point *)
    Y2 : REAL;                        (* Y-coordinate of second known point *)
    X : REAL;                         (* X-value for interpolation *)
END_VAR

VAR_OUTPUT
    Y : REAL;                         (* Interpolated Y-value *)
    Done : BOOL;                      (* Computation completed *)
    Error : BOOL;                     (* Error flag *)
    ErrorID : DWORD;                  (* Error code *)
END_VAR

VAR
    DeltaX : REAL;                    (* Difference X2 - X1 *)
    DeltaY : REAL;                    (* Difference Y2 - Y1 *)
    Slope : REAL;                     (* Slope (DeltaY / DeltaX) *)
    XDiff : REAL;                     (* Difference X - X1 *)
END_VAR

(* Initialize outputs *)
Y := 0.0;
Done := FALSE;
Error := FALSE;
ErrorID := 0;

(* Main logic *)
IF Execute THEN
    (* Validate inputs for numerical stability *)
    IF ABS(X1) > 1.0E10 OR ABS(Y1) > 1.0E10 OR 
       ABS(X2) > 1.0E10 OR ABS(Y2) > 1.0E10 OR 
       ABS(X) > 1.0E10 THEN
        Error := TRUE;
        ErrorID := 16#80010000; (* Invalid input values *)
        Y := Y1; (* Fallback: Use Y1 *)
        Done := TRUE;
        RETURN;
    END_IF;

    (* Compute differences *)
    DeltaX := X2 - X1;                (* Difference between X-coordinates *)
    XDiff := X - X1;                  (* Distance from X to X1 *)
    DeltaY := Y2 - Y1;                (* Difference between Y-coordinates *)

    (* Check for division by zero *)
    IF ABS(DeltaX) > 1.0E-10 THEN     (* Avoid division by zero or near-zero *)
        (* Calculate slope *)
        Slope := DeltaY / DeltaX;     (* Slope = (Y2 - Y1) / (X2 - X1) *)

        (* Perform linear interpolation *)
        Y := Y1 + (XDiff * Slope);    (* Y = Y1 + (X - X1) * Slope *)

        (* Check for numerical overflow in result *)
        IF ABS(Y) > 1.0E10 THEN
            Error := TRUE;
            ErrorID := 16#80020000; (* Numerical overflow *)
            Y := Y1; (* Fallback: Use Y1 *)
        END_IF;
    ELSE
        (* X1 ≈ X2, use Y1 as fallback *)
        Y := Y1;
        Error := TRUE;
        ErrorID := 16#80030000; (* X1 equals X2 *)
    END_IF;

    Done := TRUE;
ELSE
    (* Reset outputs when Execute is FALSE *)
    Y := 0.0;
    Done := FALSE;
    Error := FALSE;
    ErrorID := 0;
END_IF;

END_FUNCTION_BLOCK
