FUNCTION_BLOCK RealComparatorFB
VAR_INPUT
    Enable: BOOL;               // Enable comparison
    Input1: REAL;               // First REAL input
    Input2: REAL;               // Second REAL input
    Precision: INT := 2;        // Number of decimal places (0 to 6)
END_VAR

VAR_OUTPUT
    Equal: BOOL;                // TRUE if inputs are equal to Precision
    Greater: BOOL;              // TRUE if Input1 > Input2 to Precision
    Less: BOOL;                 // TRUE if Input1 < Input2 to Precision
    Error: BOOL;                // TRUE if invalid input (e.g., negative Precision)
END_VAR

VAR
    Scale: REAL;                // Scaling factor (10^Precision)
    Rounded1: DINT;             // Rounded Input1 as integer
    Rounded2: DINT;             // Rounded Input2 as integer
    ValidInput: BOOL;           // Input validation flag
    TempInput1: REAL;           // Temporary scaled Input1
    TempInput2: REAL;           // Temporary scaled Input2
END_VAR

// Reset outputs when disabled
IF NOT Enable THEN
    Equal := FALSE;
    Greater := FALSE;
    Less := FALSE;
    Error := FALSE;
    RETURN;
END_IF;

// Input validation
ValidInput := TRUE;
Error := FALSE;

IF Precision < 0 OR Precision > 6 THEN
    ValidInput := FALSE;
    Error := TRUE;
    Equal := FALSE;
    Greater := FALSE;
    Less := FALSE;
    RETURN;
END_IF;

// Calculate scaling factor (10^Precision)
CASE Precision OF
    0: Scale := 1.0;
    1: Scale := 10.0;
    2: Scale := 100.0;
    3: Scale := 1000.0;
    4: Scale := 10000.0;
    5: Scale := 100000.0;
    6: Scale := 1000000.0;
END_CASE;

// Perform comparison
IF ValidInput THEN
    // Scale inputs and round to nearest integer
    TempInput1 := Input1 * Scale;
    TempInput2 := Input2 * Scale;
    
    // Round to nearest integer (REAL_TO_INT rounds toward zero, so adjust for negative numbers)
    IF TempInput1 >= 0.0 THEN
        Rounded1 := REAL_TO_INT(TempInput1 + 0.5);
    ELSE
        Rounded1 := REAL_TO_INT(TempInput1 - 0.5);
    END_IF;
    
    IF TempInput2 >= 0.0 THEN
        Rounded2 := REAL_TO_INT(TempInput2 + 0.5);
    ELSE
        Rounded2 := REAL_TO_INT(TempInput2 - 0.5);
    END_IF;

    // Compare rounded values
    Equal := (Rounded1 = Rounded2);
    Greater := (Rounded1 > Rounded2);
    Less := (Rounded1 < Rounded2);
END_IF;
END_FUNCTION_BLOCK
