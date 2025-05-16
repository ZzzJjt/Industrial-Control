(* IEC 61131-3 Structured Text: Compare REAL Values with Specified Precision *)
(* Purpose: Compares two REAL inputs up to n decimal places, handling floating-point inaccuracies *)

FUNCTION_BLOCK CompareRealPrecision
VAR_INPUT
    Enable : BOOL;                  (* TRUE to initiate comparison *)
    Input1 : REAL;                  (* First REAL value to compare *)
    Input2 : REAL;                  (* Second REAL value to compare *)
    Precision : INT;                (* Number of decimal places, 0 to 6 *)
END_VAR
VAR_OUTPUT
    Equal : BOOL;                   (* TRUE if inputs are equal to specified precision *)
    Greater : BOOL;                 (* TRUE if Input1 > Input2 to specified precision *)
    Less : BOOL;                    (* TRUE if Input1 < Input2 to specified precision *)
    Error : BOOL;                   (* TRUE if Precision or inputs are invalid *)
END_VAR
VAR
    Scale : REAL;                   (* Scaling factor: 10^Precision *)
    Scaled1 : REAL;                 (* Scaled and rounded Input1 *)
    Scaled2 : REAL;                 (* Scaled and rounded Input2 *)
    MaxPrecision : INT := 6;        (* Maximum allowed precision *)
    MaxValue : REAL := 1E7;         (* Maximum absolute input value *)
END_VAR

(* Main Logic *)
IF NOT Enable THEN
    (* Reset outputs when disabled *)
    Equal := FALSE;
    Greater := FALSE;
    Less := FALSE;
    Error := FALSE;
ELSE
    (* Input Validation *)
    IF Precision < 0 OR Precision > MaxPrecision OR 
       ABS(Input1) > MaxValue OR ABS(Input2) > MaxValue THEN
        (* Invalid Precision or input values too large *)
        Error := TRUE;
        Equal := FALSE;
        Greater := FALSE;
        Less := FALSE;
    ELSE
        Error := FALSE;
        
        (* Calculate Scaling Factor *)
        Scale := 1.0;
        FOR i := 1 TO Precision DO
            Scale := Scale * 10.0;
        END_FOR;
        
        (* Scale and Round Inputs *)
        Scaled1 := ROUND(Input1 * Scale);
        Scaled2 := ROUND(Input2 * Scale);
        
        (* Compare Rounded Values *)
        IF Scaled1 = Scaled2 THEN
            Equal := TRUE;
            Greater := FALSE;
            Less := FALSE;
        ELSIF Scaled1 > Scaled2 THEN
            Equal := FALSE;
            Greater := TRUE;
            Less := FALSE;
        ELSE
            Equal := FALSE;
            Greater := FALSE;
            Less := TRUE;
        END_IF;
    END_IF;
END_IF;

(* Notes:
   - Purpose: Compares two REAL values up to n decimal places for reliable floating-point comparisons.
   - Inputs:
     - Enable: Initiates comparison when TRUE.
     - Input1, Input2: REAL values to compare.
     - Precision: INT, number of decimal places (0 to 6).
   - Outputs:
     - Equal: TRUE if inputs are equal to specified precision.
     - Greater: TRUE if Input1 > Input2 to specified precision.
     - Less: TRUE if Input1 < Input2 to specified precision.
     - Error: TRUE if Precision < 0, > 6, or |Input1|, |Input2| > 1E7.
   - Logic:
     - Scales inputs by 10^Precision, rounds to integers, and compares.
     - E.g., for Precision=2, compares ROUND(Input1 * 100) vs ROUND(Input2 * 100).
   - Safety:
     - Validates Precision (0 to 6) to prevent overflow.
     - Limits input range (|Input| <= 1E7) to avoid floating-point issues.
     - Resets outputs when Enable = FALSE for safe state.
   - Usage:
     - Ideal for process comparisons, alarming, or control switching in automation.
     - Reusable in PLC programs (e.g., CODESYS, TwinCAT) for reliable REAL comparisons.
   - Maintenance:
     - Scalar math ensures PLC compatibility.
     - Clear comments and modular design aid debugging.
     - Error flag supports diagnostics via HMI/logging.
   - Platform Notes:
     - Uses REAL for calculations; LREAL can be used for higher precision.
     - Assumes ROUND and basic floating-point operations are supported.
     - MaxPrecision=6 avoids overflow in typical 32-bit REAL (7-digit precision).
*)
END_FUNCTION_BLOCK
