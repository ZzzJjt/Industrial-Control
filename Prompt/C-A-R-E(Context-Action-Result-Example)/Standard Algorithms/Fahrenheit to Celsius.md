**Fahrenheit to Celsius:**
Implement a self-contained function block in IEC 61131-3 structured text to convert Fahrenheit to Celsius. Ensure it handles input validation and provides accurate results.

**C-A-R-E:**

🟥 C (Context) – Background Situation

Temperature conversion between Fahrenheit and Celsius is frequently needed in industrial control systems—especially in HVAC, process heating, and international deployments. Relying on manual calculations or repeated code snippets can lead to inconsistencies, rounding errors, and logic duplication. A dedicated, reusable Structured Text function block ensures accuracy, validation, and consistent integration into PLC programs.

⸻

🟩 A (Action) – Task to Perform

Create a self-contained function block in IEC 61131-3 Structured Text that:
	•	Accepts a temperature input in Fahrenheit
	•	Converts the value to Celsius using the formula:
 C = (F - 32) \times \frac{5}{9}
	•	Performs input validation, such as checking that the input is not below absolute zero (−459.67°F)
	•	Outputs the result in Celsius, along with a ValidInput boolean status flag
	•	Includes clear comments, modular structure, and scan-cycle-safe logic

⸻

🟨 R (Result) – Expected Outcome

The outcome should be a modular and reliable function block that can be reused in any program requiring Fahrenheit-to-Celsius conversion. It will offer:
	•	Accurate results using real data types
	•	Built-in validation to protect against invalid sensor readings
	•	Clean, maintainable code suitable for both educational and production PLC applications

⸻

🟦 E (Example) – Concrete Illustration

Example logic inside the function block:
IF Fahrenheit >= -459.67 THEN
   Celsius := (Fahrenheit - 32.0) * 5.0 / 9.0;
   ValidInput := TRUE;
ELSE
   Celsius := 0.0;
   ValidInput := FALSE;
END_IF;
Inputs:
	•	Fahrenheit : REAL

Outputs:
	•	Celsius : REAL
	•	ValidInput : BOOL
