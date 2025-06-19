Certainly! Below is a detailed list of all mathematical function blocks available in the OSCAT (Open Source Community for Automation Technology) library for PLC programming. The list includes descriptions, typical use cases, and categorization for easy reference.

---

# OSCAT Mathematical Function Blocks Reference Guide

## Trigonometric Functions

### SIN
- **Function Block Name**: `SIN`
- **Functional Description**: Computes the sine of an angle given in radians.
- **Typical Use Case**: Waveform generation, oscillation modeling, motion control.
- **Input/Output Types**:
  - **Input**: Angle (REAL)
  - **Output**: Sine value (REAL)

### COS
- **Function Block Name**: `COS`
- **Functional Description**: Computes the cosine of an angle given in radians.
- **Typical Use Case**: Waveform generation, oscillation modeling, motion control.
- **Input/Output Types**:
  - **Input**: Angle (REAL)
  - **Output**: Cosine value (REAL)

### TAN
- **Function Block Name**: `TAN`
- **Functional Description**: Computes the tangent of an angle given in radians.
- **Typical Use Case**: Slope calculations, angle transformations.
- **Input/Output Types**:
  - **Input**: Angle (REAL)
  - **Output**: Tangent value (REAL)

### ASIN
- **Function Block Name**: `ASIN`
- **Functional Description**: Computes the arc sine (inverse sine) of a value.
- **Typical Use Case**: Angle determination from known ratios, inverse trigonometry problems.
- **Input/Output Types**:
  - **Input**: Value (-1 to 1, REAL)
  - **Output**: Arc sine value (radians, REAL)

### ACOS
- **Function Block Name**: `ACOS`
- **Functional Description**: Computes the arc cosine (inverse cosine) of a value.
- **Typical Use Case**: Angle determination from known ratios, inverse trigonometry problems.
- **Input/Output Types**:
  - **Input**: Value (-1 to 1, REAL)
  - **Output**: Arc cosine value (radians, REAL)

### ATAN
- **Function Block Name**: `ATAN`
- **Functional Description**: Computes the arc tangent (inverse tangent) of a value.
- **Typical Use Case**: Angle determination from known ratios, slope calculations.
- **Input/Output Types**:
  - **Input**: Value (REAL)
  - **Output**: Arc tangent value (radians, REAL)

### ATAN2
- **Function Block Name**: `ATAN2`
- **Functional Description**: Computes the arc tangent of y/x using the signs of arguments to determine the correct quadrant.
- **Typical Use Case**: Angle calculation in polar coordinates, robot kinematics.
- **Input/Output Types**:
  - **Inputs**: y (REAL), x (REAL)
  - **Output**: Arc tangent value (radians, REAL)

## Algebraic Operations

### ADD
- **Function Block Name**: `ADD`
- **Functional Description**: Adds two numbers.
- **Typical Use Case**: Basic arithmetic operations, summing sensor readings.
- **Input/Output Types**:
  - **Inputs**: Number1 (REAL), Number2 (REAL)
  - **Output**: Sum (REAL)

### SUB
- **Function Block Name**: `SUB`
- **Functional Description**: Subtracts one number from another.
- **Typical Use Case**: Difference calculation, error computation.
- **Input/Output Types**:
  - **Inputs**: Minuend (REAL), Subtrahend (REAL)
  - **Output**: Difference (REAL)

### MUL
- **Function Block Name**: `MUL`
- **Functional Description**: Multiplies two numbers.
- **Typical Use Case**: Scaling factors, area/volume calculations.
- **Input/Output Types**:
  - **Inputs**: Factor1 (REAL), Factor2 (REAL)
  - **Output**: Product (REAL)

### DIV
- **Function Block Name**: `DIV`
- **Functional Description**: Divides one number by another.
- **Typical Use Case**: Rate calculations, average values.
- **Input/Output Types**:
  - **Inputs**: Dividend (REAL), Divisor (REAL)
  - **Output**: Quotient (REAL)
  - **Notes**: Division by zero is handled internally, returning a default value or triggering an alarm.

### MOD
- **Function Block Name**: `MOD`
- **Functional Description**: Computes the remainder of the division of two integers.
- **Typical Use Case**: Indexing, periodic tasks.
- **Input/Output Types**:
  - **Inputs**: Dividend (INT), Divisor (INT)
  - **Output**: Remainder (INT)

### POW
- **Function Block Name**: `POW`
- **Functional Description**: Raises a number to the power of another number.
- **Typical Use Case**: Exponential growth models, power calculations.
- **Input/Output Types**:
  - **Inputs**: Base (REAL), Exponent (REAL)
  - **Output**: Result (REAL)

### SQRT
- **Function Block Name**: `SQRT`
- **Functional Description**: Computes the square root of a number.
- **Typical Use Case**: Root mean square calculations, geometric computations.
- **Input/Output Types**:
  - **Input**: Value (REAL)
  - **Output**: Square root (REAL)

## Statistical Calculations

### MEAN_ARRAY
- **Function Block Name**: `MEAN_ARRAY`
- **Functional Description**: Computes the mean (average) of an array of numbers.
- **Typical Use Case**: Process data smoothing, quality control.
- **Input/Output Types**:
  - **Inputs**: Array (ARRAY[1..N] OF REAL), Size (INT)
  - **Output**: Mean (REAL)

### MEDIAN_ARRAY
- **Function Block Name**: `MEDIAN_ARRAY`
- **Functional Description**: Computes the median of an array of numbers.
- **Typical Use Case**: Robust statistical analysis, outlier detection.
- **Input/Output Types**:
  - **Inputs**: Array (ARRAY[1..N] OF REAL), Size (INT)
  - **Output**: Median (REAL)

### MAX_ARRAY
- **Function Block Name**: `MAX_ARRAY`
- **Functional Description**: Finds the maximum value in an array of numbers.
- **Typical Use Case**: Identifying peak values, limit checks.
- **Input/Output Types**:
  - **Inputs**: Array (ARRAY[1..N] OF REAL), Size (INT)
  - **Output**: Maximum value (REAL)

### MIN_ARRAY
- **Function Block Name**: `MIN_ARRAY`
- **Functional Description**: Finds the minimum value in an array of numbers.
- **Typical Use Case**: Identifying trough values, limit checks.
- **Input/Output Types**:
  - **Inputs**: Array (ARRAY[1..N] OF REAL), Size (INT)
  - **Output**: Minimum value (REAL)

### SUM_ARRAY
- **Function Block Name**: `SUM_ARRAY`
- **Functional Description**: Computes the sum of all elements in an array.
- **Typical Use Case**: Total accumulation, energy consumption tracking.
- **Input/Output Types**:
  - **Inputs**: Array (ARRAY[1..N] OF REAL), Size (INT)
  - **Output**: Sum (REAL)

## Exponential/Logarithmic Functions

### EXP
- **Function Block Name**: `EXP`
- **Functional Description**: Computes the exponential of a number (e^x).
- **Typical Use Case**: Growth models, probability distributions.
- **Input/Output Types**:
  - **Input**: Exponent (REAL)
  - **Output**: Exponential value (REAL)

### LN
- **Function Block Name**: `LN`
- **Functional Description**: Computes the natural logarithm (base e) of a number.
- **Typical Use Case**: Logarithmic scales, decay models.
- **Input/Output Types**:
  - **Input**: Value (>0, REAL)
  - **Output**: Natural logarithm (REAL)

### LOG10
- **Function Block Name**: `LOG10`
- **Functional Description**: Computes the base 10 logarithm of a number.
- **Typical Use Case**: pH calculations, decibel levels.
- **Input/Output Types**:
  - **Input**: Value (>0, REAL)
  - **Output**: Base 10 logarithm (REAL)

## Array and Data Processing

### SORT_ARRAY
- **Function Block Name**: `SORT_ARRAY`
- **Functional Description**: Sorts an array of numbers in ascending order.
- **Typical Use Case**: Organizing data, preparing for analysis.
- **Input/Output Types**:
  - **Inputs**: Array (ARRAY[1..N] OF REAL), Size (INT)
  - **Outputs**: SortedArray (ARRAY[1..N] OF REAL)

### FILTER_LOW_PASS
- **Function Block Name**: `FILTER_LOW_PASS`
- **Functional Description**: Applies a low-pass filter to smooth out high-frequency noise in a signal.
- **Typical Use Case**: Signal filtering, process data smoothing.
- **Input/Output Types**:
  - **Inputs**: InputSignal (REAL), FilterConstant (REAL)
  - **Output**: FilteredSignal (REAL)

### FILTER_HIGH_PASS
- **Function Block Name**: `FILTER_HIGH_PASS`
- **Functional Description**: Applies a high-pass filter to remove low-frequency noise in a signal.
- **Typical Use Case**: Noise reduction, signal enhancement.
- **Input/Output Types**:
  - **Inputs**: InputSignal (REAL), FilterConstant (REAL)
  - **Output**: FilteredSignal (REAL)

### INTERPOLATE_LINEAR
- **Function Block Name**: `INTERPOLATE_LINEAR`
- **Functional Description**: Performs linear interpolation between two points.
- **Typical Use Case**: Interpolating sensor data, generating lookup tables.
- **Input/Output Types**:
  - **Inputs**: X1 (REAL), Y1 (REAL), X2 (REAL), Y2 (REAL), X (REAL)
  - **Output**: Y (REAL)

### LIMIT_VALUE
- **Function Block Name**: `LIMIT_VALUE`
- **Functional Description**: Limits a value within a specified range.
- **Typical Use Case**: Ensuring sensor values remain within safe limits, preventing overflows.
- **Input/Output Types**:
  - **Inputs**: Value (REAL), LowerLimit (REAL), UpperLimit (REAL)
  - **Output**: LimitedValue (REAL)

### CLIP_VALUE
- **Function Block Name**: `CLIP_VALUE`
- **Functional Description**: Clips a value to a specified range, similar to limiting but with different handling.
- **Typical Use Case**: Clamping actuator positions, ensuring valid ranges.
- **Input/Output Types**:
  - **Inputs**: Value (REAL), LowerClip (REAL), UpperClip (REAL)
  - **Output**: ClippedValue (REAL)

### SCALE_VALUE
- **Function Block Name**: `SCALE_VALUE`
- **Functional Description**: Scales a value from one range to another.
- **Typical Use Case**: Converting sensor outputs to engineering units, adjusting control signals.
- **Input/Output Types**:
  - **Inputs**: Value (REAL), FromMin (REAL), FromMax (REAL), ToMin (REAL), ToMax (REAL)
  - **Output**: ScaledValue (REAL)

---

## Notes on Usage Constraints and Special Features

- **Precision**: Most functions operate with REAL data types, providing precision suitable for industrial applications.
- **Range Checks**: Some functions perform internal range checks to prevent invalid inputs, such as division by zero or negative logarithms.
- **Performance**: Efficient implementation ensures minimal computational overhead, suitable for real-time applications.
- **Interoperability**: Designed to work seamlessly with other OSCAT function blocks and standard IEC 61131-3 data types.

## Conclusion

This comprehensive list serves as a quick-reference guide for developers working with the OSCAT library. It provides detailed descriptions and typical use cases for each mathematical function block, helping engineers efficiently implement and maintain PLC programs. This document can be included in engineering handbooks, project documentation, or internal wikis, and serves as a valuable training resource for teams adopting OSCAT in automation projects.

---

Feel free to customize or expand this guide based on specific project needs or additional function blocks found in the latest version of the OSCAT library.
