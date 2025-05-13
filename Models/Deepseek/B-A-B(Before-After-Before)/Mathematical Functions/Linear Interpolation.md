FUNCTION_BLOCK LinearInterpolation
{ SFC: Linear interpolation between two points }
{ Author: Automation Engineer }
{ Date: 2025-05-13 }

(*
    Description:
    Performs linear interpolation between two known points (X1,Y1) and (X2,Y2)
    at a given X-value.

    Formula:
    Y = Y1 + ((X - X1) * (Y2 - Y1)) / (X2 - X1)

    Edge Case Handling:
    If X1 == X2 (vertical line), returns Y1 to avoid division by zero.
    This can be extended with diagnostics or alarms if desired.

    Precision Notes:
    - Uses REAL arithmetic; floating point inaccuracies may occur
    - Avoid extremely small differences between X2 and X1 to prevent instability
*)

VAR_INPUT
    X1 : REAL;   // X-coordinate of first point
    Y1 : REAL;   // Y-coordinate of first point
    X2 : REAL;   // X-coordinate of second point
    Y2 : REAL;   // Y-coordinate of second point
    X  : REAL;   // Target X-value for interpolation
END_VAR

VAR_OUTPUT
    Y : REAL;    // Interpolated Y-value
END_VAR

VAR
    deltaX : REAL := 0.0;
    deltaY : REAL := 0.0;
    diffX : REAL := 0.0;
END_VAR

// Compute difference between X2 and X1
diffX := X2 - X1;

// Avoid division by zero
IF ABS(diffX) > 1.0E-9 THEN
    // Compute slope components
    deltaX := X - X1;
    deltaY := Y2 - Y1;

    // Apply linear interpolation formula
    Y := Y1 + (deltaX * deltaY) / diffX;
ELSE
    // If X2 == X1, return Y1 (no unique Y due to vertical line)
    Y := Y1;

    // Optional: Raise an error flag or diagnostic signal here if needed
END_IF;

END_FUNCTION_BLOCK

PROGRAM PLC_PRG
VAR
    LinInterp1: LinearInterpolation;
    SensorInputRAW: REAL := 450.0;     // Raw sensor value
    SensorMinRAW: REAL := 0.0;         // Min raw value
    SensorMaxRAW: REAL := 1023.0;      // Max raw value
    SensorMinENG: REAL := 0.0;         // Engineering min
    SensorMaxENG: REAL := 100.0;       // Engineering max
    ProcessValue: REAL := 0.0;
END_VAR

LinInterp1(
    X1 := SensorMinRAW,
    Y1 := SensorMinENG,
    X2 := SensorMaxRAW,
    Y2 := SensorMaxENG,
    X  := SensorInputRAW
);

ProcessValue := LinInterp1.Y;
