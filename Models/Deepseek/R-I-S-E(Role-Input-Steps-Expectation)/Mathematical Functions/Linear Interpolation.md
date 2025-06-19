FUNCTION_BLOCK FB_LinearInterpolation
VAR_INPUT
    X1 : REAL := 0.0;     // First known x-value
    Y1 : REAL := 0.0;     // First known y-value
    X2 : REAL := 1.0;     // Second known x-value
    Y2 : REAL := 1.0;     // Second known y-value
    X  : REAL := 0.5;     // Query point between X1 and X2
END_VAR

VAR_OUTPUT
    Y : REAL := 0.0;      // Interpolated result
END_VAR

VAR
    delta_x : REAL := 0.0;
    delta_y : REAL := 0.0;
    slope   : REAL := 0.0;
END_VAR

// Compute differences
delta_x := X2 - X1;
delta_y := Y2 - Y1;

// Safety check: prevent division by zero
IF ABS(delta_x) > 1.0E-6 THEN
    // Calculate slope and interpolate using:
    // Y = Y1 + ((X - X1) * (Y2 - Y1)) / (X2 - X1)
    slope := delta_y / delta_x;
    Y := Y1 + slope * (X - X1);
ELSE
    // If X1 == X2, return Y1 (no unique line can be formed)
    Y := Y1;
END_IF;

PROGRAM PLC_PRG
VAR
    Interpolator: FB_LinearInterpolation;

    SensorInputRAW: REAL := 1234.0;     // Raw sensor input (e.g., 4–20 mA scaled to 0–10000)
    ProcessValue: REAL := 0.0;          // Scaled value (e.g., 0–100 %)

    // Example: Convert 4mA = 0%, 20mA = 100%
    RawMin: REAL := 4000.0;             // 4 mA = 4000 counts
    RawMax: REAL := 20000.0;            // 20 mA = 20000 counts
    ScaleMin: REAL := 0.0;
    ScaleMax: REAL := 100.0;
END_VAR

// Perform linear interpolation from raw to scaled
Interpolator(
    X1 := RawMin,
    Y1 := ScaleMin,
    X2 := RawMax,
    Y2 := ScaleMax,
    X := SensorInputRAW
);

ProcessValue := Interpolator.Y;
