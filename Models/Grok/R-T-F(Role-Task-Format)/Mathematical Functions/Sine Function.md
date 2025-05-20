FUNCTION_BLOCK SineFunction
VAR_INPUT
    AngleRad : REAL; (* Input angle in radians *)
END_VAR
VAR_OUTPUT
    SineValue : REAL; (* Computed sine value *)
    ErrorCode : INT := 0; (* 0: Success, 1: Invalid input *)
END_VAR
VAR
    NormalizedAngle : REAL; (* Angle normalized to [-π, π] *)
    x, x2, x3, x5, x7 : REAL; (* Powers for Taylor series *)
    PI : REAL := 3.141592653589793; (* Constant for π *)
    TwoPI : REAL := 6.283185307179586; (* Constant for 2π *)
    USE_BUILTIN_SIN : BOOL := TRUE; (* Flag to simulate SIN() availability *)
END_VAR

(* Initialize outputs *)
SineValue := 0.0;
ErrorCode := 0;

(* Validate input *)
IF NOT IS_VALID_REAL(AngleRad) THEN
    (* Input must be finite (not NaN or infinity) *)
    SineValue := 0.0;
    ErrorCode := 1; (* Invalid input *)
    RETURN;
END_IF;

(* Normalize angle to [-π, π] to improve accuracy *)
(* sin(x) = sin(x + 2πk), so reduce x mod 2π *)
NormalizedAngle := AngleRad;
WHILE NormalizedAngle > PI DO
    NormalizedAngle := NormalizedAngle - TwoPI;
END_WHILE;
WHILE NormalizedAngle < -PI DO
    NormalizedAngle := NormalizedAngle + TwoPI;
END_WHILE;

(* Compute sine *)
IF USE_BUILTIN_SIN THEN
    (* Use built-in SIN() function for optimal performance *)
    (* SIN() computes sin(x) directly, where x is in radians *)
    SineValue := SIN(NormalizedAngle);
ELSE
    (* Use Taylor series approximation: sin(x) ≈ x - x^3/3! + x^5/5! - x^7/7! *)
    (* Terms up to x^7 provide good accuracy for x in [-π, π] *)
    x := NormalizedAngle;
    x2 := x * x; (* x^2 *)
    x3 := x2 * x; (* x^3 *)
    x5 := x3 * x2; (* x^5 *)
    x7 := x5 * x2; (* x^7 *)
    
    (* Compute series: x - x^3/6 + x^5/120 - x^7/5040 *)
    (* Factorials: 3! = 6, 5! = 120, 7! = 5040 *)
    SineValue := x - (x3 / 6.0) + (x5 / 120.0) - (x7 / 5040.0);
END_IF;

(* Validate result *)
IF NOT IS_VALID_REAL(SineValue) THEN
    SineValue := 0.0;
    ErrorCode := 1; (* Invalid result *)
END_IF;

(* Helper function to check if a REAL value is valid *)
METHOD IS_VALID_REAL : BOOL
VAR_INPUT
    Value : REAL;
END_VAR
IS_VALID_REAL := NOT (Value = 0.0 / 0.0) AND NOT (Value = 1.0 / 0.0);
END_METHOD

END_FUNCTION_BLOCK
