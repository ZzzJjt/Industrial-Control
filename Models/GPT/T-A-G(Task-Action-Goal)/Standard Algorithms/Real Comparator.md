FUNCTION_BLOCK FB_RealCompare
VAR_INPUT
    Input1    : REAL;   // First REAL input value
    Input2    : REAL;   // Second REAL input value
    Precision : INT;    // Number of decimal places to compare
    Enable    : BOOL;   // Execution control
END_VAR

VAR_OUTPUT
    Equal     : BOOL := FALSE;   // TRUE if rounded values are equal
    Greater   : BOOL := FALSE;   // TRUE if Input1 > Input2
    Less      : BOOL := FALSE;   // TRUE if Input1 < Input2
    Error     : BOOL := FALSE;   // TRUE if invalid Precision or disabled
END_VAR

VAR
    Scale     : REAL;
    I1        : DINT;
    I2        : DINT;
END_VAR

// ============ Main Logic ============

IF Enable THEN
    IF Precision >= 0 THEN
        Error := FALSE;

        // Compute scaling factor: Scale := 10^Precision
        Scale := EXP(Precision * LN(10.0));

        // Round and convert to integers
        I1 := DINT(ROUND(Input1 * Scale));
        I2 := DINT(ROUND(Input2 * Scale));

        // Reset all flags
        Equal := FALSE;
        Greater := FALSE;
        Less := FALSE;

        // Compare values
        IF I1 = I2 THEN
            Equal := TRUE;
        ELSIF I1 > I2 THEN
            Greater := TRUE;
        ELSE
            Less := TRUE;
        END_IF

    ELSE
        // Invalid precision
        Error := TRUE;
        Equal := FALSE;
        Greater := FALSE;
        Less := FALSE;
    END_IF
ELSE
    // Disabled
    Error := TRUE;
    Equal := FALSE;
    Greater := FALSE;
    Less := FALSE;
END_IF
