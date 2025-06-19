FUNCTION_BLOCK FB_RealComparator
VAR_INPUT
    Enable    : BOOL;      // Start comparison
    Input1    : REAL;      // First input value
    Input2    : REAL;      // Second input value
    Precision : INT;       // Number of decimal places (0..6 recommended)
END_VAR

VAR_OUTPUT
    Equal     : BOOL;      // Input1 == Input2 (within precision)
    Greater   : BOOL;      // Input1 > Input2
    Less      : BOOL;      // Input1 < Input2
    Error     : BOOL;      // TRUE if invalid Precision input
END_VAR

VAR
    Scale        : REAL;
    Scaled1      : DINT;
    Scaled2      : DINT;
    ValidInputs  : BOOL;
END_VAR

// Input validation
IF Enable THEN
    Error := (Precision < 0) OR (Precision > 9); // Limit precision to avoid overflow
    ValidInputs := NOT Error;

    IF ValidInputs THEN
        // Calculate scaling factor: 10^Precision
        Scale := 1.0;
        FOR i := 1 TO Precision DO
            Scale := Scale * 10.0;
        END_FOR;

        // Scale and round both inputs
        Scaled1 := REAL_TO_DINT(Input1 * Scale);
        Scaled2 := REAL_TO_DINT(Input2 * Scale);

        // Perform comparison
        Equal := (Scaled1 = Scaled2);
        Greater := (Scaled1 > Scaled2);
        Less := (Scaled1 < Scaled2);
    ELSE
        // Invalidate all results if error
        Equal := FALSE;
        Greater := FALSE;
        Less := FALSE;
    END_IF;
END_IF;
