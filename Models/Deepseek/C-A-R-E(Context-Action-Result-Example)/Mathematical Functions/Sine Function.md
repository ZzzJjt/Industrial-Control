FUNCTION_BLOCK SineFunction_TaylorSeries
VAR_INPUT
    AngleRad : REAL; // Input angle in radians
END_VAR

VAR_OUTPUT
    SineValue : REAL; // Output sine value
END_VAR

VAR
    Term : REAL; // Current term in the Taylor series
    Sum : REAL; // Accumulated sum of the Taylor series
    Factorial : REAL; // Computed factorial for each term
    Power : REAL; // Computed power for each term
    i : INT; // Loop index
    Sign : INT; // Sign alternator for the series (+1, -1)
END_VAR

// Initialize variables
SineValue := 0.0;
Term := 0.0;
Sum := 0.0;
Factorial := 1.0;
Power := 1.0;
Sign := 1;

// Normalize the angle to [-π, π] to improve accuracy
AngleRad := AngleRad - 2.0 * #PI * TRUNC((AngleRad + #PI) / (2.0 * #PI));

// Taylor series expansion for sine: sin(x) = x - x^3/3! + x^5/5! - x^7/7! + ...
FOR i := 0 TO 4 DO // Adjust the number of terms for desired precision
    // Calculate the current term: (-1)^i * x^(2i+1) / (2i+1)!
    Term := Sign * Power / Factorial;
    Sum := Sum + Term;
    
    // Update for next iteration
    Sign := Sign * -1; // Alternate sign
    Power := Power * AngleRad * AngleRad; // Increment power by x^2
    Factorial := Factorial * (2*i + 2) * (2*i + 3); // Increment factorial
END_FOR;

SineValue := Sum;

// Inline comments explaining the logic:
// The sine function computes the sine of an angle in radians using a Taylor series approximation.
// The input domain is expected to be in radians.
// The angle is normalized to [-π, π] to improve accuracy.
// The Taylor series is truncated after 5 terms for balance between accuracy and performance.
// No recursion or deep nesting is used to ensure deterministic execution.
// The approximation may introduce small errors for very large angles due to truncation.
// Precision constraints of REAL types are considered, but care is taken to minimize overflow.



