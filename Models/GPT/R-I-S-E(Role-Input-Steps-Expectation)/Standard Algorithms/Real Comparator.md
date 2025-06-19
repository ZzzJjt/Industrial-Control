FUNCTION_BLOCK FB_RealComparator
VAR_INPUT
    Enable    : BOOL;
    Input1    : REAL;
    Input2    : REAL;
    Precision : INT; // Number of decimal places to compare
END_VAR

VAR_OUTPUT
    Equal     : BOOL := FALSE;
    Greater   : BOOL := FALSE;
    Less      : BOOL := FALSE;
    Error     : BOOL := FALSE;
END_VAR

VAR
    Scale        : REAL;
    Scaled1      : DINT;
    Scaled2      : DINT;
    iPrecisionOK : BOOL;
END_VAR

IF Enable THEN
    // Reset outputs
    Equal := FALSE;
    Greater := FALSE;
    Less := FALSE;
    Error := FALSE;

    // Validate precision
    iPrecisionOK := (Precision >= 0 AND Precision <= 9); // Avoid overflow

    IF iPrecisionOK THEN
        // Compute scale factor: 10^Precision
        Scale := EXP(Precision * LN(10.0));

        // Scale and round inputs
        Scaled1 := REAL_TO_DINT(Input1 * Scale + 0.5); // Add 0.5 to round
        Scaled2 := REAL_TO_DINT(Input2 * Scale + 0.5);

        // Compare
        IF Scaled1 = Scaled2 THEN
            Equal := TRUE;
        ELSIF Scaled1 > Scaled2 THEN
            Greater := TRUE;
        ELSE
            Less := TRUE;
        END_IF
    ELSE
        Error := TRUE;
    END_IF
ELSE
    // Disabled
    Equal := FALSE;
    Greater := FALSE;
    Less := FALSE;
    Error := TRUE;
END_IF
