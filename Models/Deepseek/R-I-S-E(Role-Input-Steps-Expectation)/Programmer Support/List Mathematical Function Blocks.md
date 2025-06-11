# 📘 **OSCAT Mathematical Function Blocks – Quick Reference Guide**

This guide organizes the most commonly used **mathematical function blocks** from the **OSCAT (Open Source Controller Automation Toolkit)** library. It is designed to help PLC developers and automation engineers quickly locate and apply the right function block for control, data processing, or industrial math tasks.

---

## 🟩 Categories of OSCAT Math Function Blocks

| Category | Description |
|---------|-------------|
| 🔺 Trigonometric | Functions like sine, cosine, tangent, and their inverses |
| 📈 Exponential & Logarithmic | Powers, roots, exponentials, logarithms |
| 📊 Statistical | Averaging, standard deviation, min/max tracking |
| ➕ Algebraic | Basic arithmetic operations, sign handling, absolute values |

---

## 🟨 1. 🔺 Trigonometric Functions

| Block Name | Description | Use Case | Notes |
|------------|-------------|----------|-------|
| `SIN` | Computes sine of an angle in radians | Angle-based position control | Input in radians |
| `COS` | Computes cosine of an angle in radians | Motion trajectory planning | Input in radians |
| `TAN` | Computes tangent of an angle in radians | Inclination sensing | Avoid near π/2 |
| `ASIN` | Inverse sine (arcsine), returns angle in radians | Sensor angle reconstruction | Input range: [-1, 1] |
| `ACOS` | Inverse cosine (arccosine), returns angle in radians | Geometry calculations | Input range: [-1, 1] |
| `ATAN` | Inverse tangent (arctangent), returns angle in radians | Direction detection | Handles full circle |
| `ATAN2` | Inverse tangent with two inputs, returns angle in radians | Vector direction calculation | Useful for X/Y coordinates |
| `DEG_TO_RAD` | Converts degrees to radians | Interface with HMI or sensors | Multiply by π/180 internally |
| `RAD_TO_DEG` | Converts radians to degrees | Display angles on HMI | Multiply by 180/π internally |

---

## 🟦 2. 📈 Exponential & Logarithmic Functions

| Block Name | Description | Use Case | Notes |
|------------|-------------|----------|-------|
| `EXP` | Natural exponential function e^x | Growth modeling, PID tuning | x can be any REAL |
| `LN` | Natural logarithm | Decay processes, thermodynamics | x > 0 required |
| `LOG` | Base-10 logarithm | Signal dB conversion, pH calculations | x > 0 required |
| `POW` | Power function a^b | Mechanical stress calculations | Accepts real exponents |
| `SQRT` | Square root | Velocity from kinetic energy, RMS | Input ≥ 0 |
| `CBRT` | Cube root | Volume-to-dimension conversions | Supports negative input |
| `HYPOT` | Hypotenuse of two values (sqrt(a² + b²)) | Distance between points | Prevents overflow |
| `LOGX` | Logarithm with arbitrary base | Custom scaling functions | Base ≠ 1 and > 0 |

---

## 🟧 3. 📊 Statistical Functions

| Block Name | Description | Use Case | Notes |
|------------|-------------|----------|-------|
| `MEAN` | Calculates mean (average) of N values | Process variable monitoring | Typically uses array input |
| `STDEV` | Standard deviation of a set of samples | Quality control, variance analysis | Requires at least 2 samples |
| `MIN` | Returns minimum value from a list | Fault detection (e.g., low temp) | Works on arrays |
| `MAX` | Returns maximum value from a list | Overload detection (e.g., pressure peak) | Works on arrays |
| `LIMIT` | Clamps value between lower and upper bounds | Safety limits enforcement | Output = MIN(MAX(input, lo), hi) |
| `RMS` | Root Mean Square of a sequence | AC signal analysis, vibration | Common in power systems |
| `MEDIAN` | Median value of a dataset | Noise filtering in sensor data | Slower than MEAN |
| `SUM` | Sum of a list of numbers | Batch totalization, counters | Can be used with timers |

---

## 🟥 4. ➕ Algebraic Functions

| Block Name | Description | Use Case | Notes |
|------------|-------------|----------|-------|
| `ABS` | Absolute value | Error magnitude regardless of sign | Always returns positive |
| `SIGN` | Returns -1, 0, or +1 based on sign | Direction logic | Useful in motor control |
| `FLOOR` | Rounds down to nearest integer | Bucketing analog values | E.g., binning temperatures |
| `CEIL` | Rounds up to nearest integer | Resource allocation | Memory or buffer size |
| `TRUNC` | Truncates decimal part | Integer conversion without rounding | Preserves sign |
| `FRAC` | Extracts fractional part | Fine-tuning adjustments | Useful in PID outputs |
| `MOD` | Modulo operation | Cyclic behavior, indexing | Only for integers |
| `DIV` | Integer division | Loop counters, batch steps | Discards remainder |
| `CLAMP` | Limits value between min and max | Safety boundaries | Same as LIMIT |
| `LIN_INTERPOLATE` | Linear interpolation between two points | Scaling sensor inputs | Y = mX + b equivalent |

---

## 🟪 Optional Enhancements

Would you like this reference expanded with:
- Parameter tables showing inputs/outputs?
- Example code snippets?
- IEC 61131-3 language syntax notes?
- Integration examples with specific PLC platforms (e.g., Codesys, Beckhoff)?

Let me know — I can tailor this into a downloadable PDF or internal wiki page for your team!
