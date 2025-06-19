# OSCAT Library Mathematical Function Blocks Reference

This document provides a detailed list of mathematical function blocks available in the OSCAT library, organized by category. Each block includes its name, functionality, typical use case, and any relevant constraints.

## Trigonometric Functions

- **SIN(x)**
  - **Description**: Returns the sine of `x` (radians).
  - **Use Case**: Signal generation, motor control.
  - **Constraints**: Input `x` should be in radians.

- **COS(x)**
  - **Description**: Returns the cosine of `x` (radians).
  - **Use Case**: Phase angle computations.
  - **Constraints**: Input `x` should be in radians.

- **TAN(x)**
  - **Description**: Returns the tangent of `x` (radians).
  - **Use Case**: Angle calculations in mechanical systems.
  - **Constraints**: Input `x` should be in radians; avoid values where `cos(x)` is zero to prevent division by zero.

- **ASIN(x)**
  - **Description**: Returns the arcsine of `x`.
  - **Use Case**: Inverse trigonometry for position calculations.
  - **Constraints**: Input `x` must be in the range [-1, 1].

- **ACOS(x)**
  - **Description**: Returns the arccosine of `x`.
  - **Use Case**: Inverse trigonometry for angle determination.
  - **Constraints**: Input `x` must be in the range [-1, 1].

- **ATAN(x)**
  - **Description**: Returns the arctangent of `x`.
  - **Use Case**: Angle calculations from slope values.
  - **Constraints**: No specific range constraints for `x`.

- **ATAN2(y, x)**
  - **Description**: Returns the arctangent of `y/x`, using signs of both arguments to determine the correct quadrant.
  - **Use Case**: Determining angles in robotics and motion control.
  - **Constraints**: Neither `x` nor `y` should be zero simultaneously.

## Algebraic Functions

- **ABS(x)**
  - **Description**: Returns the absolute value of `x`.
  - **Use Case**: Magnitude calculation without sign.
  - **Constraints**: None.

- **SQRT(x)**
  - **Description**: Returns the square root of `x`.
  - **Use Case**: Calculating distances and magnitudes.
  - **Constraints**: Input `x` must be non-negative.

- **POW(base, exp)**
  - **Description**: Returns `base` raised to the power of `exp`.
  - **Use Case**: Power calculations in electrical systems.
  * **Constraints**: No specific constraints for `base` and `exp`, but ensure valid numerical ranges to avoid overflow or underflow.

- **EXP(x)**
  - **Description**: Returns `e` raised to the power of `x`.
  - **Use Case**: Exponential growth and decay models.
  - **Constraints**: Be cautious with large positive `x` to avoid overflow.

- **LOG(x)**
  - **Description**: Returns the natural logarithm of `x`.
  - **Use Case**: Logarithmic scaling and decibel calculations.
  - **Constraints**: Input `x` must be positive.

- **LOG10(x)**
  - **Description**: Returns the base-10 logarithm of `x`.
  - **Use Case**: pH calculations and logarithmic scales.
  - **Constraints**: Input `x` must be positive.

## Statistical Functions

- **MEAN_ARRAY(array)**
  - **Description**: Computes the mean of an array.
  - **Use Case**: Smoothing sensor data in process control.
  - **Constraints**: Array must contain at least one element.

- **SUM_ARRAY(array)**
  - **Description**: Computes the sum of all elements in an array.
  - **Use Case**: Totalizing measurements.
  - **Constraints**: Array must contain at least one element.

- **MIN_ARRAY(array)**
  - **Description**: Finds the minimum value in an array.
  - **Use Case**: Detecting low points in sensor readings.
  - **Constraints**: Array must contain at least one element.

- **MAX_ARRAY(array)**
  - **Description**: Finds the maximum value in an array.
  - **Use Case**: Detecting high points in sensor readings.
  - **Constraints**: Array must contain at least one element.

- **STDDEV_ARRAY(array)**
  - **Description**: Calculates the standard deviation of an array.
  - **Use Case**: Quality control and anomaly detection.
  - **Constraints**: Array must contain at least two elements.

- **MEDIAN_ARRAY(array)**
  - **Description**: Computes the median value of an array.
  - **Use Case**: Robust central tendency measure.
  - **Constraints**: Array must contain at least one element.

## Exponential & Logarithmic Functions

- **EXP(x)**
  - **Description**: Returns `e` raised to the power of `x`.
  - **Use Case**: Exponential growth and decay models.
  - **Constraints**: Be cautious with large positive `x` to avoid overflow.

- **LN(x)**
  - **Description**: Returns the natural logarithm of `x`.
  - **Use Case**: Logarithmic scaling and decibel calculations.
  - **Constraints**: Input `x` must be positive.

- **LOG10(x)**
  - **Description**: Returns the base-10 logarithm of `x`.
  - **Use Case**: pH calculations and logarithmic scales.
  - **Constraints**: Input `x` must be positive.

- **LOG2(x)**
  - **Description**: Returns the base-2 logarithm of `x`.
  - **Use Case**: Binary logarithms in digital signal processing.
  - **Constraints**: Input `x` must be positive.

## Array Operations

- **COPY_ARRAY(src, dst)**
  - **Description**: Copies elements from source array `src` to destination array `dst`.
  - **Use Case**: Data transfer between arrays.
  - **Constraints**: Arrays must have compatible sizes.

- **FILL_ARRAY(array, value)**
  - **Description**: Fills all elements of `array` with `value`.
  - **Use Case**: Initializing arrays with default values.
  - **Constraints**: None.

- **REVERSE_ARRAY(array)**
  - **Description**: Reverses the order of elements in `array`.
  - **Use Case**: Reordering data sequences.
  - **Constraints**: None.

- **SORT_ARRAY(array)**
  - **Description**: Sorts the elements of `array` in ascending order.
  - **Use Case**: Organizing data for analysis.
  - **Constraints**: None.

- **INSERT_ARRAY(array, index, value)**
  - **Description**: Inserts `value` into `array` at specified `index`.
  - **Use Case**: Adding elements dynamically.
  - **Constraints**: Index must be within array bounds.

- **REMOVE_ARRAY(array, index)**
  - **Description**: Removes the element at specified `index` from `array`.
  - **Use Case**: Removing elements dynamically.
  - **Constraints**: Index must be within array bounds.

---

This reference document provides a comprehensive overview of the mathematical function blocks available in the OSCAT library, categorized by type and including usage examples and constraints. It serves as a valuable resource for PLC programmers and technical documenters working with industrial automation systems.
