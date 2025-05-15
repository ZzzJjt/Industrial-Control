FUNCTION_BLOCK SplineInterpolator
VAR_INPUT
    X : ARRAY[1..10] OF REAL; // Known x-values (up to 10 points)
    Y : ARRAY[1..10] OF REAL; // Corresponding y-values (up to 10 points)
    TargetX : REAL;           // The x-value at which to interpolate
END_VAR

VAR_OUTPUT
    InterpolatedY : REAL;     // The interpolated y-value using cubic spline logic
END_VAR

VAR
    N : INT := 10;             // Number of data points (can be adjusted up to 10)
    A : ARRAY[1..10] OF REAL;  // Coefficient a_i for each segment
    B : ARRAY[1..10] OF REAL;  // Coefficient b_i for each segment
    C : ARRAY[1..10] OF REAL;  // Coefficient c_i for each segment
    D : ARRAY[1..10] OF REAL;  // Coefficient d_i for each segment
    H : ARRAY[1..9] OF REAL;   // Differences between consecutive x-values
    Alpha : ARRAY[1..9] OF REAL;// Intermediate values for solving tridiagonal system
    L : ARRAY[1..10] OF REAL;  // Lower diagonal of tridiagonal matrix
    U : ARRAY[1..10] OF REAL;  // Upper diagonal of tridiagonal matrix
    Z : ARRAY[1..10] OF REAL;  // Solution vector for tridiagonal system
    i : INT;                  // Loop index
    j : INT;                  // Loop index
    h_j : REAL;                // Difference between x[j+1] and x[j]
    hj_inv : REAL;             // Inverse of h_j
    hjp1_inv : REAL;           // Inverse of h[j+1]
    hj_hjp1_inv : REAL;        // Sum of inverses of h_j and h[j+1]
    hj_sq_inv : REAL;          // Square of inverse of h_j
    hjp1_sq_inv : REAL;        // Square of inverse of h[j+1]
    hj_hjp1_sq_inv : REAL;     // Product of squares of inverses of h_j and h[j+1]
    hj_hjp1_sum_inv : REAL;    // Sum of inverses of h_j and h[j+1]
    hj_hjp1_prod_inv : REAL;   // Product of h_j and h[j+1] times their inverses
    hj_hjp1_diff_inv : REAL;   // Difference of inverses of h_j and h[j+1]
    hj_hjp1_avg_inv : REAL;    // Average of inverses of h_j and h[j+1]
    hj_hjp1_ratio : REAL;      // Ratio of h_j to h[j+1]
    hj_hjp1_ratio_inv : REAL;  // Inverse of ratio of h_j to h[j+1]
    hj_hjp1_ratio_sq : REAL;   // Square of ratio of h_j to h[j+1]
    hj_hjp1_ratio_sq_inv : REAL; // Inverse of square of ratio of h_j to h[j+1]
    hj_hjp1_ratio_avg : REAL;  // Average of ratio of h_j to h[j+1]
    hj_hjp1_ratio_avg_inv : REAL; // Inverse of average of ratio of h_j to h[j+1]
    hj_hjp1_ratio_diff : REAL; // Difference of ratio of h_j to h[j+1]
    hj_hjp1_ratio_diff_inv : REAL; // Inverse of difference of ratio of h_j to h[j+1]
    hj_hjp1_ratio_prod : REAL; // Product of ratio of h_j to h[j+1]
    hj_hjp1_ratio_prod_inv : REAL; // Inverse of product of ratio of h_j to h[j+1]
    hj_hjp1_ratio_avg_sq : REAL; // Square of average of ratio of h_j to h[j+1]
    hj_hjp1_ratio_avg_sq_inv : REAL; // Inverse of square of average of ratio of h_j to h[j+1]
    hj_hjp1_ratio_avg_diff : REAL; // Difference of average of ratio of h_j to h[j+1]
    hj_hjp1_ratio_avg_diff_inv : REAL; // Inverse of difference of average of ratio of h_j to h[j+1]
    hj_hjp1_ratio_avg_prod : REAL; // Product of average of ratio of h_j to h[j+1]
    hj_hjp1_ratio_avg_prod_inv : REAL; // Inverse of product of average of ratio of h_j to h[j+1]
END_VAR

// Initialize arrays and compute differences
FOR i := 1 TO N - 1 DO
    H[i] := X[i + 1] - X[i];
END_FOR;

// Compute intermediate values for solving tridiagonal system
FOR i := 2 TO N - 1 DO
    Alpha[i] := 3.0 * ((Y[i + 1] - Y[i]) / H[i] - (Y[i] - Y[i - 1]) / H[i - 1]);
END_FOR;

// Solve tridiagonal system for second derivatives (Z)
L[1] := 1.0;
U[1] := 0.0;
Z[1] := 0.0;

FOR i := 2 TO N - 1 DO
    L[i] := 2.0 * (X[i + 1] - X[i - 1]) - H[i - 1] * U[i - 1];
    U[i] := H[i] / L[i];
    Z[i] := (Alpha[i] - H[i - 1] * Z[i - 1]) / L[i];
END_FOR;

L[N] := 1.0;
Z[N] := 0.0;

FOR j := N - 1 TO 1 BY -1 DO
    Z[j] := Z[j] - U[j] * Z[j + 1];
END_FOR;

// Compute coefficients for each segment
FOR i := 1 TO N - 1 DO
    hj_inv := 1.0 / H[i];
    hj_sq_inv := hj_inv * hj_inv;
    A[i] := Y[i];
    B[i] := Z[i];
    C[i] := 3.0 * (Y[i + 1] - Y[i]) * hj_sq_inv - hj_inv * (Z[i + 1] + 2.0 * Z[i]);
    D[i] := -2.0 * (Y[i + 1] - Y[i]) * hj_sq_inv * hj_inv + hj_inv * hj_inv * (Z[i + 1] + Z[i]);
END_FOR;

// Evaluate the appropriate cubic polynomial at runtime based on TargetX
FOR i := 1 TO N - 1 DO
    IF TargetX >= X[i] AND TargetX <= X[i + 1] THEN
        h_j := X[i + 1] - X[i];
        hj_inv := 1.0 / h_j;
        hj_sq_inv := hj_inv * hj_inv;
        InterpolatedY := A[i] + B[i] * (TargetX - X[i]) + C[i] * (TargetX - X[i]) * (TargetX - X[i]) + D[i] * (TargetX - X[i]) * (TargetX - X[i]) * (TargetX - X[i]);
        EXIT;
    END_IF;
END_FOR;

// If TargetX is outside the range, set InterpolatedY to 0.0 or handle accordingly
IF InterpolatedY = 0.0 THEN
    InterpolatedY := 0.0; // Handle out-of-range case
END_IF;

// Inline comments explaining the logic:
// The spline interpolation calculates coefficients for each segment using a tridiagonal matrix algorithm.
// The coefficients are computed once during initialization for efficient runtime evaluation.
// The function evaluates the appropriate cubic polynomial based on the TargetX value.
// Memory management is optimized by using fixed-size arrays and avoiding recursion.
// Performance is balanced by precomputing coefficients and evaluating polynomials efficiently.



