# OSCAT Library Mathematical Function Blocks Reference

## Introduction

The OSCAT (Open Source Community for Automation Technology) library provides a comprehensive set of open-source function blocks (FBs) and functions (FCs) compliant with IEC 61131-3, designed for industrial automation. The OSCAT BASIC library, in particular, includes a robust mathematics section with functions tailored for control applications. This document serves as a reference guide for all mathematical function blocks and functions in the OSCAT BASIC library (version 3.33), focusing on their functionality, typical industrial use cases, and constraints. It is organized into categories for easy navigation and is suitable for inclusion in manuals, onboarding kits, or engineering wikis.

The OSCAT BASIC library contains approximately 65 mathematical functions, as noted in community discussions, covering trigonometric, algebraic, statistical, exponential, logarithmic, and array operations. Each function is implemented as a function (FC) rather than a function block (FB), meaning they are stateless and return a single value without internal memory. The library is vendor-independent, tested across platforms like CODESYS, TwinCAT, and PCWorx, and available for free download from the CODESYS Store or OSCAT website.[](https://store.codesys.com/en/oscat-basic.html)[](https://www.tecomat.com/download/software-and-firmware/mosaic/oscat-basic/)[](https://openplc.discussion.community/post/oscat-library-maths-section-10256390)

## Mathematical Function Blocks by Category

Below is a detailed list of mathematical functions in the OSCAT BASIC library, grouped by category. Each entry includes the function name, description, industrial use case, and constraints where applicable. Note that OSCAT functions are typically implemented as FCs (not FBs), so they do not maintain internal state and are invoked per scan cycle.

### Trigonometric Functions

| Function Name | Description | Use Case | Constraints |
|---------------|-------------|----------|-------------|
| `SIN`         | Returns the sine of an angle (input in radians). | Generating sinusoidal signals for motor control or simulating oscillatory processes (e.g., in HVAC systems). | Input: REAL (radians). Output: REAL (-1.0 to 1.0). Ensure input is finite to avoid NaN. |
| `COS`         | Returns the cosine of an angle (input in radians). | Calculating phase angles in AC power systems or positioning in robotic arms. | Input: REAL (radians). Output: REAL (-1.0 to 1.0). Finite input required. |
| `TAN`         | Returns the tangent of an angle (input in radians). | Slope calculations in conveyor belt control or angle-based navigation systems. | Input: REAL (radians, avoid π/2 ± nπ to prevent overflow). Output: REAL. |
| `ASIN`        | Returns the arcsine (inverse sine) of a value, in radians. | Converting normalized sensor data (e.g., -1.0 to 1.0) to angular positions in motion control. | Input: REAL (-1.0 to 1.0). Output: REAL (-π/2 to π/2 radians). |
| `ACOS`        | Returns the arccosine (inverse cosine) of a value, in radians. | Determining angular offsets in servo systems or antenna alignment. | Input: REAL (-1.0 to 1.0). Output: REAL (0 to π radians). |
| `ATAN`        | Returns the arctangent (inverse tangent) of a value, in radians. | Calculating angles from slope data in material handling or crane control. | Input: REAL. Output: REAL (-π/2 to π/2 radians). |
| `ATAN2`       | Returns the arctangent of y/x, considering quadrant, in radians. | Resolving angular positions in 2D coordinate systems (e.g., robotic arm positioning). | Inputs: REAL (y, x). Output: REAL (-π to π radians). Avoid x = y = 0. |

**Constraints**:
- All trigonometric functions expect inputs in radians, not degrees. Use `RAD` or `DEG` conversion functions if needed.
- Ensure inputs are within valid ranges to avoid undefined results (e.g., `TAN` at π/2).
- Functions return REAL values, limited by 32-bit floating-point precision (~7 digits).

### Algebraic Functions

| Function Name | Description | Use Case | Constraints |
|---------------|-------------|----------|-------------|
| `ABS`         | Returns the absolute value of a number. | Normalizing sensor errors or calculating deviations in process control (e.g., pressure differences). | Input: REAL. Output: REAL (non-negative). |
| `SQRT`        | Returns the square root of a non-negative number. | Calculating RMS values in power monitoring or distance in positioning systems. | Input: REAL (≥0.0). Output: REAL. Negative inputs return NaN. |
| `SQR`         | Returns the square of a number (x²). | Computing power (P = I²R) in electrical systems or scaling sensor signals. | Input: REAL. Output: REAL. Large inputs may cause overflow. |
| `MIN`         | Returns the minimum of two values. | Selecting the lowest sensor reading in redundant systems (e.g., temperature sensors). | Inputs: REAL (two values). Output: REAL. |
| `MAX`         | Returns the maximum of two values. | Determining peak values in flow or pressure monitoring. | Inputs: REAL (two values). Output: REAL. |
| `LIMIT`       | Clamps a value between a minimum and maximum. | Restricting control outputs (e.g., valve position 0–100%) to safe ranges. | Inputs: REAL (value, min, max). Output: REAL. Ensure min ≤ max. |
| `MOD`         | Returns the modulus (remainder) of x/y. | Implementing cyclic counters or scheduling in batch processes. | Inputs: REAL (x, y). Output: REAL. Avoid y = 0. |

**Constraints**:
- Functions operate on REAL inputs/outputs, requiring type conversion for INT or DINT inputs.
- Check for division by zero (`MOD`) or invalid inputs (`SQRT`) to avoid runtime errors.
- `LIMIT` assumes `min ≤ max`; validate inputs externally if necessary.

### Statistical Functions

| Function Name | Description | Use Case | Constraints |
|---------------|-------------|----------|-------------|
| `MEAN`        | Computes the mean of two values. | Averaging redundant sensor readings (e.g., pressure sensors) for smoother control. | Inputs: REAL (two values). Output: REAL. |
| `ARRAY_MEAN`  | Computes the mean of an array of values. | Smoothing time-series sensor data in process monitoring (e.g., temperature trends). | Input: ARRAY OF REAL. Output: REAL. Non-empty array required. |
| `ARRAY_SDV`   | Calculates the standard deviation of an array. | Detecting anomalies in quality control (e.g., product weight variations). | Input: ARRAY OF REAL (min 2 elements). Output: REAL. Requires sufficient elements. |
| `ARRAY_MIN`   | Returns the minimum value in an array. | Identifying the lowest sensor reading in a multi-point system (e.g., tank levels). | Input: ARRAY OF REAL. Output: REAL. Non-empty array required. |
| `ARRAY_MAX`   | Returns the maximum value in an array. | Monitoring peak values in a batch process (e.g., maximum pressure). | Input: ARRAY OF REAL. Output: REAL. Non-empty array required. |
| `ARRAY_SUM`   | Computes the sum of an array’s values. | Aggregating flow rates in multi-stream processes (e.g., chemical dosing). | Input: ARRAY OF REAL. Output: REAL. Non-empty array required. |

**Constraints**:
- Array functions require non-empty arrays and sufficient elements (e.g., `ARRAY_SDV` needs ≥2 elements for variance calculation).
- Array size must be defined at compile time; dynamic resizing is not supported.
- Outputs are REAL, subject to 32-bit floating-point limitations.[](https://forge.codesys.com/forge/talk/Engineering/thread/5886148e49/)

### Exponential & Logarithmic Functions

| Function Name | Description | Use Case | Constraints |
|---------------|-------------|----------|-------------|
| `EXP`         | Returns e^x (exponential function). | Modeling growth/decay in chemical processes (e.g., reaction rates). | Input: REAL. Output: REAL. Large inputs may cause overflow. |
| `LN`          | Returns the natural logarithm of a number. | Linearizing sensor data (e.g., pH sensors) or calculating decay constants. | Input: REAL (>0.0). Output: REAL. Non-positive inputs return NaN. |
| `LOG`         | Returns the base-10 logarithm of a number. | Scaling large datasets in process monitoring (e.g., flow rates). | Input: REAL (>0.0). Output: REAL. Non-positive inputs return NaN. |
| `POW`         | Returns x^y (x raised to power y). | Calculating non-linear relationships (e.g., pump power vs. flow). | Inputs: REAL (x, y). Output: REAL. Avoid x = 0 with negative y. |

**Constraints**:
- Ensure positive inputs for `LN` and `LOG` to avoid undefined results.
- `POW` may overflow for large exponents; validate inputs for process ranges.
- Functions assume REAL inputs/outputs, requiring conversion for other types.

### Array Operations

| Function Name | Description | Use Case | Constraints |
|---------------|-------------|----------|-------------|
| `ARRAY_SORT`  | Sorts an array in ascending order. | Ordering sensor data for analysis (e.g., prioritizing fault conditions). | Input: ARRAY OF REAL. Output: ARRAY OF REAL. Non-empty array required. |
| `ARRAY_REVERSE`| Reverses the order of an array’s elements. | Reordering data for reverse processing (e.g., batch sequence analysis). | Input: ARRAY OF REAL. Output: ARRAY OF REAL. Non-empty array required. |
| `ARRAY_FILL`  | Fills an array with a specified value. | Initializing data buffers for process logging (e.g., zeroing temperature logs). | Inputs: ARRAY OF REAL, REAL (value). Non-empty array required. |
| `ARRAY_COPY`  | Copies elements from one array to another. | Backing up sensor data before processing (e.g., flow rate logs). | Inputs: ARRAY OF REAL (source, destination). Arrays must be compatible. |

**Constraints**:
- Array operations require matching array sizes for source and destination (`ARRAY_COPY`).
- Non-empty arrays are mandatory; validate array bounds at runtime.
- Operations modify arrays in-place, so ensure data is backed up if needed.

## Additional Notes
- **Library Access**: The OSCAT BASIC library is available for free from the CODESYS Store (`https://store.codesys.com/oscat-basic.html`) or OSCAT website (`http://www.oscat.de`). It requires manual installation in environments like CODESYS, TwinCAT, or Mosaic.[](https://store.codesys.com/en/oscat-basic.html)[](https://www.tecomat.com/download/software-and-firmware/mosaic/oscat-basic/)
- **Documentation**: Detailed English and German documentation is provided for each function, accessible via the OSCAT website or included with the library download.[](https://www.tecomat.com/download/software-and-firmware/mosaic/oscat-basic/)[](https://openplc.discussion.community/post/oscat-library-maths-section-10256390)
- **Constraints**:
  - All functions use REAL (32-bit floating-point) inputs/outputs, requiring type conversion for INT, DINT, or other types.
  - Functions are stateless (no internal memory), executed per scan cycle, and do not support instantiation as objects (e.g., `ARRAY_SDV` cannot be instantiated like an FB).[](https://forge.codesys.com/forge/talk/Engineering/thread/5886148e49/)
  - Ensure inputs are finite and within valid ranges to avoid NaN or overflow errors, as OSCAT functions do not include built-in input validation.
- **Use in Industrial Automation**:
  - These functions are ideal for tasks like signal processing, data analysis, and control calculations in manufacturing, chemical processing, and HVAC systems.
  - Example: Use `ARRAY_MEAN` and `ARRAY_SDV` to monitor and stabilize temperature in a reactor, or `SIN` and `COS` for generating waveforms in motor control.
- **Portability**: The library is vendor-independent, tested on CODESYS, TwinCAT, PCWorx, and others, but may require reference adjustments (e.g., `Tc2_Standard` in TwinCAT).[](https://www.motioncontroltips.com/what-are-oscat-files-how-do-they-relate-to-codesys/)[](https://alltwincat.com/2018/01/01/oscat-basic-library-in-twincat/)

## Conclusion

The OSCAT BASIC library’s mathematical functions provide a powerful toolkit for industrial automation, offering trigonometric, algebraic, statistical, exponential, logarithmic, and array operations. Each function is designed for portability and efficiency, supporting applications from signal generation to process monitoring. By understanding their functionality, use cases, and constraints, engineers can leverage these functions to enhance control logic, improve data processing, and ensure robust system performance. This reference document serves as a concise guide for integrating OSCAT’s mathematical tools into PLC projects, streamlining development and maintenance.
