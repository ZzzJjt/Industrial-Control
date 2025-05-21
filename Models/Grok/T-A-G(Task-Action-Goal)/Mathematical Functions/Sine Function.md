(* Function Block: Sine Calculator *)
(* Version: 1.0, Date: 2025-05-21 *)
FUNCTION_BLOCK FB_SineFunction
VAR_INPUT
    Execute : BOOL;                   (* Trigger computation *)
    AngleRad : REAL;                  (* Input angle in radians *)
END_VAR

VAR_OUTPUT
    SineValue : REAL;                 (* Computed sine of the input angle *)
    Done : BOOL;                      (* Computation completed *)
    Error : BOOL;                     (* Error flag *)
    ErrorID : DWORD;                  (* Error code: 0x8001xxxx = Input error, 0x8002xxxx = Numerical *)
END_VAR

VAR
    NormalizedAngle : REAL;           (* Normalized angle for computation *)
    TWO_PI : REAL := 6.283185307179586; (* 2π constant for angle normalization *)
END_VAR

(* Initialize outputs *)
SineValue := 0.0;
Done := FALSE;
Error := FALSE;
ErrorID := 0;

(* Main logic *)
IF Execute THEN
    (* Validate input for numerical stability *)
    (* Large angles can cause precision loss in SIN() due to floating-point limitations *)
    IF ABS(AngleRad) > 1.0E10 THEN
        Error := TRUE;
        ErrorID := 16#80010001;       (* Input angle too large *)
        SineValue := 0.0;             (* Fallback to zero for safe control *)
        Done := TRUE;
        RETURN;
    END_IF;

    (* Normalize angle to [-π, π] to improve numerical accuracy *)
    (* SIN() is periodic with period 2π, so AngleRad = AngleRad mod 2π *)
    (* This reduces precision loss for large angles *)
    NormalizedAngle := AngleRad - TWO_PI * TRUNC(AngleRad / TWO_PI);
    IF NormalizedAngle > 3.141592653589793 THEN
        NormalizedAngle := NormalizedAngle - TWO_PI;
    ELSIF NormalizedAngle < -3.141592653589793 THEN
        NormalizedAngle := NormalizedAngle + TWO_PI;
    END_IF;

    (* Compute sine using built-in SIN function *)
    (* SIN() expects radians and returns sin(x) in [-1, 1] *)
    (* Radians are used as they are the standard for mathematical libraries *)
    SineValue := SIN(NormalizedAngle);

    (* Check result for numerical validity *)
    (* SIN() should return values in [-1, 1]; anything else indicates an error *)
    IF ABS(SineValue) > 1.0 OR ABS(SineValue) > 1.0E10 THEN
        Error := TRUE;
        ErrorID := 16#80020001;       (* Invalid sine result *)
        SineValue := 0.0;             (* Fallback to zero *)
    END_IF;

    Done := TRUE;                     (* Mark computation as complete *)
ELSE
    (* Reset outputs when Execute is FALSE *)
    (* Ensures deterministic behavior between scans *)
    SineValue := 0.0;
    Done := FALSE;
    Error := FALSE;
    ErrorID := 0;
END_IF;

END_FUNCTION_BLOCK
