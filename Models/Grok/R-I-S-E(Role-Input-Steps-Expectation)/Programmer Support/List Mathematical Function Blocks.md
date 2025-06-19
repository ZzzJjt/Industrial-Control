OSCAT Mathematical Function Blocks Reference
This reference organizes mathematical function blocks (FBs) from the OSCAT BASIC library, designed for IEC 61131-3-compliant PLCs. It aims to help automation engineers quickly select appropriate blocks, understand their functionality, and apply them in industrial control systems. Each entry includes the block name, a functional description, a typical use case, and notes on parameters or limitations. The blocks are categorized into Trigonometric, Exponential & Logarithmic, Statistical, and Algebraic groups.
Trigonometric Function Blocks



Name
Description
Use Case
Notes/Limitations



SIN_FB
Computes the sine of an input angle, maintaining state for continuous updates.
Calculating oscillatory motion in a robotic arm’s angular positioning.
Input: Angle in radians (REAL). Output: REAL (-1.0 to 1.0). Ensure input is normalized to avoid overflow.


COS_FB
Computes the cosine of an input angle, with internal memory for tracking.
Determining phase shifts in a conveyor belt’s cyclic motion control.
Input: Angle in radians (REAL). Output: REAL (-1.0 to 1.0). Avoid rapid input changes to reduce noise.


TAN_FB
Computes the tangent of an input angle, suitable for angular control loops.
Adjusting blade angles in a wind turbine based on wind direction.
Input: Angle in radians (REAL). Output: REAL. Avoid inputs near ±π/2 to prevent undefined results.


ARCTAN_FB
Computes the arctangent of an input ratio, used for inverse trigonometric tasks.
Converting Cartesian coordinates to polar angles in a CNC machine.
Input: Ratio (REAL). Output: Angle in radians (REAL, -π/2 to π/2). Check input range for stability.


Exponential & Logarithmic Function Blocks



Name
Description
Use Case
Notes/Limitations



EXP_FB
Calculates the exponential (e^x) of an input, with state for iterative use.
Modeling heat transfer rates in a chemical reactor’s temperature control.
Input: REAL. Output: REAL (>0). Large inputs may cause overflow; clamp if needed.


LN_FB
Computes the natural logarithm of an input, suitable for growth/decay models.
Estimating decay rates in a wastewater treatment’s chemical dosing system.
Input: REAL (>0). Output: REAL. Ensure positive input to avoid undefined results.


LOG10_FB
Computes the base-10 logarithm of an input, used for scaling measurements.
Converting sensor data (e.g., pH levels) to logarithmic scales in a lab.
Input: REAL (>0). Output: REAL. Validate input to prevent errors.


Statistical Function Blocks



Name
Description
Use Case
Notes/Limitations



AVG_FB
Calculates the running average of a stream of input values, with memory.
Smoothing flow rate measurements in a pipeline to reduce sensor noise.
Input: REAL. Output: REAL. Parameter: Sample size (INT). Larger samples increase latency.


STDDEV_FB
Computes the standard deviation of input values over a window, for variability analysis.
Monitoring product weight consistency on a packaging line.
Input: REAL. Output: REAL. Parameter: Window size (INT). Ensure sufficient samples for accuracy.


MINMAX_FB
Tracks minimum and maximum values of an input stream, with reset capability.
Recording peak temperatures in a furnace for quality control.
Input: REAL. Outputs: Min, Max (REAL). Reset via BOOL input. Memory-intensive for long runs.


Algebraic Function Blocks



Name
Description
Use Case
Notes/Limitations



ABS_FB
Computes the absolute value of an input, maintaining state for continuous use.
Ensuring positive pressure readings in a gas storage tank control system.
Input: REAL. Output: REAL (≥0). No significant limitations; robust for most inputs.


SQRT_FB
Calculates the square root of an input, used in geometric or power calculations.
Determining conveyor belt speed based on squared sensor inputs.
Input: REAL (≥0). Output: REAL (≥0). Ensure non-negative input to avoid errors.


POLY_FB
Evaluates a polynomial function of an input, configurable with coefficients.
Modeling nonlinear pump curves in a water treatment system.
Inputs: X (REAL), Coefficients (ARRAY OF REAL). Output: REAL. Validate array size.


Usage Notes

Instantiation: Each FB requires a unique instance name declared in the POU (e.g., VAR MySin: SIN_FB; END_VAR), as FBs maintain internal memory.
Parameter Types: Most inputs/outputs are REAL for floating-point precision. Check OSCAT documentation for specific INT or DINT requirements.
Limitations: Some FBs (e.g., TAN_FB, LN_FB) have input restrictions to avoid undefined results. Always validate inputs in the PLC logic.
Units: Trigonometric FBs expect radians; others assume unitless or context-specific units (e.g., bar for pressure). Align with process requirements.
Testing: OSCAT FBs are extensively tested across platforms (e.g., CODESYS, TwinCAT), but verify functionality in your specific PLC environment, as no formal support is guaranteed.

Conclusion
This reference organizes OSCAT’s mathematical function blocks into intuitive categories, providing clear descriptions and practical use cases to streamline development. It serves as a training tool for new engineers and a quick lookup for experienced developers, reducing reliance on source code searches. By selecting the appropriate block based on its description and limitations, teams can enhance code reliability and efficiency in control systems, from chemical processing to manufacturing automation. For detailed parameter lists or additional blocks, consult the OSCAT BASIC documentation at oscat.de or the CODESYS Store.
