```
FUNCTION_BLOCK SplineInterpolation
VAR_INPUT
    X : ARRAY [1..N] OF REAL; // X-coordinates of data points
    Y : ARRAY [1..N] OF REAL; // Y-coordinates of data points
    XQuery : REAL; // X-coordinate for which to interpolate Y
VAR_OUTPUT
    YInterpolated : REAL; // Interpolated Y-value
VAR
    N : INT := CNT(X); // Number of data points
    h : ARRAY [1..N-1] OF REAL; // Differences between consecutive X-values
    alpha : ARRAY [1..N-1] OF REAL; // Intermediate values for spline coefficients
    z : ARRAY [1..N] OF REAL; // Coefficients for second derivatives
    u : ARRAY [1..N-1] OF REAL; // Intermediate values for solving tridiagonal system
    y2 : ARRAY [1..N] OF REAL; // Second derivatives of the spline at knots
    i : INT; // Index for finding the correct segment
    a, b, c, d : REAL; // Coefficients of the cubic polynomial
END_VAR

// Compute the differences between consecutive X-values
FOR i := 1 TO N-1 DO
    h[i] := X[i+1] - X[i];
END_FOR;

// Compute alpha values
FOR i := 1 TO N-1 DO
    alpha[i] := (Y[i+1] - Y[i]) / h[i] - (Y[i] - Y[i-1]) / h[i-1];
END_FOR;

// Solve the tridiagonal system for z values
z[1] := 0; // Set the boundary conditions
z[N] := 0; // Set the boundary conditions
FOR i := 1 TO N-1 DO
    u[i] := 2 * (h[i] + h[i-1]);
    z[i+1] := (alpha[i] - h[i-1] * z[i]) / (u[i] - h[i-1] * z[i]);
END_FOR;

// Compute the second derivatives
FOR i := N-1 DOWNTO 1 DO
    y2[i] := z[i] - h[i-1] * z[i+1];
END_FOR;
y2[1] := 0; // Boundary condition
y2[N] := 0; // Boundary condition

// Find the correct segment and compute the interpolated value
i := 1;
WHILE XQuery > X[i+1] DO
    i := i + 1;
END_WHILE;

a := Y[i];
b := y2[i];
c := (Y[i+1] - Y[i]) / h[i] - h[i] * (y2[i] + 2 * y2[i+1]) / 3;
d := (y2[i] - y2[i+1]) / (3 * h[i]);

YInterpolated := (a + b * (XQuery - X[i]) + c * POWER((XQuery - X[i]), 2) + d * POWER((XQuery - X[i]), 3));

END_FUNCTION_BLOCK
```
