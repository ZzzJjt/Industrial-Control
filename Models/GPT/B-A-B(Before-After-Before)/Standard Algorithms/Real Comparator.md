FUNCTION_BLOCK FB_CompareRealWithPrecision
VAR_INPUT
    Enable    : BOOL;
    Input1    : REAL;
    Input2    : REAL;
    Precision : INT;  // e.g., 2 for comparison up to 0.01
END_VAR

VAR_OUTPUT
    Equal     : BOOL;
    Greater   : BOOL;
    Less      : BOOL;
    Error     : BOOL;
END_VAR

VAR
    ScaleFactor : REAL;
    Int1        : DINT;
    Int2        : DINT;
END_VAR

IF Enable THEN
    // Input validation
    IF Precision < 0 OR Precision > 9 THEN
        Error := TRUE;
        Equal := FALSE;
        Greater := FALSE;
        Less := FALSE;
    ELSE
        Error := FALSE;

        // Compute scaling factor (10^Precision)
        ScaleFactor := 1.0;
        FOR VAR i : INT := 1 TO Precision DO
            ScaleFactor := ScaleFactor * 10.0;
        END_FOR

        // Scale and round inputs
        Int1 := REAL_TO_DINT(ROUND(Input1 * ScaleFactor));
        Int2 := REAL_TO_DINT(ROUND(Input2 * ScaleFactor));

        // Perform comparison
        Equal := (Int1 = Int2);
        Greater := (Int1 > Int2);
        Less := (Int1 < Int2);
    END_IF
ELSE
    // Reset outputs if not enabled
    Equal := FALSE;
    Greater := FALSE;
    Less := FALSE;
    Error := FALSE;
END_IF
