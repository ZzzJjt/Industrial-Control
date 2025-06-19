FUNCTION_BLOCK FB_SplineInterpolator
VAR_INPUT
    // Known data points
    X : ARRAY[1..MAX_POINTS] OF REAL; // x-values (must be strictly increasing)
    Y : ARRAY[1..MAX_POINTS] OF REAL; // y-values
    CoeffA : ARRAY[1..MAX_SEGMENTS] OF REAL; // Spline coefficient a_i
    CoeffB : ARRAY[1..MAX_SEGMENTS] OF REAL; // Spline coefficient b_i
    CoeffC : ARRAY[1..MAX_SEGMENTS] OF REAL; // Spline coefficient c_i
    CoeffD : ARRAY[1..MAX_SEGMENTS] OF REAL; // Spline coefficient d_i

    TargetX : REAL; // x-value to interpolate
END_VAR

VAR_OUTPUT
    InterpolatedY : REAL := 0.0;
    Error : BOOL := FALSE; // TRUE if an error occurred
    ErrorCode : INT := 0; // 0 = OK, 1 = Too few points, 2 = Invalid index, 3 = Unsorted X
END_VAR

VAR
    i : INT := 1;
    N : INT := MAX_POINTS; // Total number of points
    foundIndex : BOOL := FALSE;
    h : REAL := 0.0;
END_VAR

// Initialize output
InterpolatedY := 0.0;
Error := FALSE;
ErrorCode := 0;

// Check for minimum required points
IF N < 2 THEN
    Error := TRUE;
    ErrorCode := 1; // Too few points
    EXIT;
END_IF;

// Ensure X array is sorted in ascending order
FOR i := 1 TO N - 1 DO
    IF X[i] >= X[i + 1] THEN
        Error := TRUE;
        ErrorCode := 3; // X not strictly increasing
        EXIT;
    END_IF;
END_FOR;

// Clamp TargetX to valid range
IF TargetX <= X[1] THEN
    InterpolatedY := Y[1];
    EXIT;
ELSIF TargetX >= X[N] THEN
    InterpolatedY := Y[N];
    EXIT;
END_IF;

// Find the segment where TargetX lies
FOR i := 1 TO N - 1 DO
    IF TargetX >= X[i] AND TargetX <= X[i + 1] THEN
        h := TargetX - X[i]; // Distance from X[i]
        InterpolatedY := CoeffA[i] + CoeffB[i] * h + CoeffC[i] * h * h + CoeffD[i] * h * h * h;
        foundIndex := TRUE;
        EXIT;
    END_IF;
END_FOR;

// Safety check
IF NOT foundIndex THEN
    Error := TRUE;
    ErrorCode := 2; // No matching segment found (should not happen)
END_IF;

PROGRAM PLC_PRG
VAR
    SplineCalc : FB_SplineInterpolator;

    PointsX : ARRAY[1..5] OF REAL := [0.0, 1.0, 2.0, 3.0, 4.0];
    PointsY : ARRAY[1..5] OF REAL := [0.0, 1.0, 0.0, -1.0, 0.0];

    A_Coeffs : ARRAY[1..4] OF REAL := [0.0, 1.0, 0.0, -1.0];
    B_Coeffs : ARRAY[1..4] OF REAL := [1.0, -2.0, -3.0, 2.0];
    C_Coeffs : ARRAY[1..4] OF REAL := [0.0, 0.0, 0.0, 0.0];
    D_Coeffs : ARRAY[1..4] OF REAL := [0.0, 0.0, 0.0, 0.0];

    InputX : REAL := 2.5;
    OutputY : REAL := 0.0;
    HasError : BOOL := FALSE;
END_VAR

// Evaluate spline
SplineCalc(
    X := PointsX,
    Y := PointsY,
    CoeffA := A_Coeffs,
    CoeffB := B_Coeffs,
    CoeffC := C_Coeffs,
    CoeffD := D_Coeffs,
    TargetX := InputX
);

OutputY := SplineCalc.InterpolatedY;
HasError := SplineCalc.Error;
