FUNCTION_BLOCK LinearInterpolation
VAR_INPUT
    X1 : REAL;   // First known X-coordinate
    Y1 : REAL;   // Corresponding Y at X1
    X2 : REAL;   // Second known X-coordinate
    Y2 : REAL;   // Corresponding Y at X2
    X  : REAL;   // Input X for which Y is to be interpolated
END_VAR
VAR_OUTPUT
    Y : REAL;    // Interpolated Y output
END_VAR
VAR
    DeltaX : REAL; // Difference X2 - X1
    DeltaY : REAL; // Difference Y2 - Y1
BEGIN
    // Step 1: Compute delta X and delta Y
    DeltaX := X2 - X1;
    DeltaY := Y2 - Y1;

    // Step 2: Check for divide-by-zero condition
    IF DeltaX <> 0.0 THEN
        // Step 3: Apply linear interpolation formula
        // Y := Y1 + ((X - X1) * (Y2 - Y1)) / (X2 - X1)
        Y := Y1 + ((X - X1) * DeltaY) / DeltaX;
    ELSE
        // Step 4: Fallback if X1 = X2 (undefined slope)
        Y := Y1; // Default to Y1 as safe fallback
    END_IF
END_FUNCTION_BLOCK
