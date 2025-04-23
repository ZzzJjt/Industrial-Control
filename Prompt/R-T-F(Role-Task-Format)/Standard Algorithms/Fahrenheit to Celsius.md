**Fahrenheit to Celsius:**
Implement a self-contained function block in IEC 61131-3 structured text to convert Fahrenheit to Celsius. Ensure it handles input validation and provides accurate results.

**R-T-F:**

🟥 R (Role) – Define Your Role

Act as a PLC engineer responsible for building utility function blocks in IEC 61131-3 Structured Text for use in industrial temperature control systems.

⸻

🟩 T (Task) – Define the Objective

Develop a self-contained function block that converts a Fahrenheit temperature value to Celsius, ensuring:
	•	Accurate conversion using the formula:
Celsius = (Fahrenheit - 32) \times \frac{5}{9}
	•	Input validation to reject values below absolute zero (−459.67°F)
	•	Scan-cycle safety and compatibility with industrial applications
	•	Clear output signaling, including a ValidInput boolean flag

⸻

🟧 F (Format) – Specify the Output Format

Provide:
	•	A complete IEC 61131-3 Structured Text function block with:
	•	Input: Fahrenheit : REAL
	•	Outputs: Celsius : REAL, ValidInput : BOOL
	•	Logic structure that includes:
	•	An IF condition to check for valid input
	•	The Celsius conversion calculation
	•	Fallback values when the input is invalid
	•	Inline comments to explain validation, conversion logic, and best practices for reuse
