OSCAT Library Mathematical Function Blocks and Functions Reference
This document provides a comprehensive list of mathematical function blocks and key mathematical functions from the OSCAT BASIC library (version 3.33) for IEC 61131-3 PLC programming. Each entry includes a description, typical use cases, and notes on parameters or limitations. The OSCAT library is open-source, vendor-independent, and designed for portability across IEC 61131-3 compliant PLCs, making it ideal for automation projects requiring advanced mathematical operations.
Categories

Algebraic Functions: Operations for basic arithmetic, scaling, and transformations.
Trigonometric Functions: Calculations involving angles and periodic signals.
Statistical Functions: Aggregations and analyses of data sets.
Special Functions: Advanced mathematical operations like logarithms and exponentials.

Mathematical Function Blocks and Functions
Algebraic Functions



Name
Type
Description
Use Cases
Notes



LIMITER
Function Block
Clamps an input value between specified upper and lower bounds.
Restricting control signals (e.g., valve positions, motor speeds) to safe ranges in PID loops or actuator control.
Inputs: IN (REAL), MIN (REAL), MAX (REAL). Output: OUT (REAL). Ensure MIN ≤ MAX to avoid undefined behavior.


SCALE
Function Block
Linearly scales an input value from one range to another.
Converting sensor readings (e.g., 4–20 mA to 0–100%) or normalizing data for HMI displays.
Inputs: IN (REAL), IN_MIN (REAL), IN_MAX (REAL), OUT_MIN (REAL), OUT_MAX (REAL). Output: OUT (REAL). Avoid IN_MAX = IN_MIN to prevent division by zero.


ABS
Function
Returns the absolute value of an input.
Ensuring positive values in calculations, such as error magnitudes in control loops.
Input: IN (REAL). Output: OUT (REAL). No range limitations.


MUL_DIV
Function
Performs multiplication and division in one operation (OUT = (IN1 * IN2) / IN3).
Efficiently computing ratios or scaled products in flow or power calculations.
Inputs: IN1, IN2, IN3 (REAL). Output: OUT (REAL). IN3 must not be zero.


Trigonometric Functions



Name
Type
Description
Use Cases
Notes



SIN
Function
Calculates the sine of an angle in radians.
Modeling periodic signals in vibration analysis or motor control.
Input: IN (REAL, radians). Output: OUT (REAL, -1.0 to 1.0). Use DEG_TO_RAD for degree inputs.


COS
Function
Calculates the cosine of an angle in radians.
Phase calculations in AC power systems or trajectory planning.
Input: IN (REAL, radians). Output: OUT (REAL, -1.0 to 1.0).


TAN
Function
Calculates the tangent of an angle in radians.
Slope calculations in motion control or geometric transformations.
Input: IN (REAL, radians, avoid ±π/2). Output: OUT (REAL). Undefined at odd multiples of π/2.


DEG_TO_RAD
Function
Converts degrees to radians.
Preprocessing angle inputs for trigonometric functions in user interfaces accepting degrees.
Input: IN (REAL, degrees). Output: OUT (REAL, radians). No limitations.


Statistical Functions



Name
Type
Description
Use Cases
Notes



AVERAGE
Function Block
Computes the running average of a stream of input values.
Smoothing sensor data (e.g., temperature, pressure) to reduce noise in control systems.
Inputs: IN (REAL), N (INT, number of samples). Output: OUT (REAL). N must be positive; large N increases memory usage.


RMS
Function Block
Calculates the root mean square of a set of values.
Analyzing AC signals or power consumption in electrical systems.
Inputs: IN (REAL), N (INT). Output: OUT (REAL). Requires multiple samples; N > 0.


STDDEV
Function Block
Computes the standard deviation of a data set.
Quality control to monitor process variability (e.g., product dimensions).
Inputs: IN (REAL), N (INT). Output: OUT (REAL). N ≥ 2 for meaningful results.


MIN_MAX
Function Block
Tracks minimum and maximum values of a data stream.
Monitoring process limits (e.g., peak temperatures) for alarms or diagnostics.
Inputs: IN (REAL). Outputs: MIN_OUT (REAL), MAX_OUT (REAL). Reset via RST (BOOL).


Special Functions



Name
Type
Description
Use Cases
Notes



LN
Function
Calculates the natural logarithm of an input.
Modeling exponential processes (e.g., chemical reaction rates).
Input: IN (REAL, > 0). Output: OUT (REAL). Undefined for IN ≤ 0.


LOG
Function
Calculates the base-10 logarithm of an input.
Scaling data for logarithmic displays or pH calculations.
Input: IN (REAL, > 0). Output: OUT (REAL). Undefined for IN ≤ 0.


EXP
Function
Computes e raised to the power of the input.
Modeling growth processes or compensating for exponential sensor responses.
Input: IN (REAL). Output: OUT (REAL). Large IN may cause overflow.


SQRT
Function
Calculates the square root of an input.
Distance calculations or normalizing squared sensor data.
Input: IN (REAL, ≥ 0). Output: OUT (REAL). Undefined for IN < 0.


Notes for Developers

Library Access: The OSCAT BASIC library is available for download from oscat.de or as a CODESYS port from the CODESYS Store. Documentation in English and German is included with version 3.33.
Function vs. Function Block: Functions (e.g., SIN, ABS) are stateless and return a single value, ideal for simple calculations. Function blocks (e.g., AVERAGE, LIMITER) maintain internal state, suitable for operations requiring memory or multiple outputs.
Usage Tips:
Ensure input validation to avoid errors (e.g., non-zero divisors, positive inputs for logarithms).
Test function blocks with realistic data sets, as some (e.g., RMS, STDDEV) require multiple samples for accuracy.
Use OSCAT’s forum (oscat.de) for community support, as no formal support is guaranteed.


Limitations:
OSCAT libraries are open-source and extensively tested but carry no functionality guarantee. Validate implementations in critical applications.
Some function blocks (e.g., AVERAGE, RMS) may consume significant memory for large sample sizes.


Integration: Compatible with CODESYS, TwinCAT, PCWorx, and other IEC 61131-3 environments. Import the library manually in some IDEs (e.g., Mosaic).

References

OSCAT Official Website: www.oscat.de
OSCAT BASIC Library on CODESYS Store: store.codesys.com
Motion Control Tips on OSCAT Libraries: www.motioncontroltips.com

This reference serves as a quick guide for control engineers integrating OSCAT’s mathematical function blocks and functions into automation projects, enhancing precision and efficiency in industrial control systems.
