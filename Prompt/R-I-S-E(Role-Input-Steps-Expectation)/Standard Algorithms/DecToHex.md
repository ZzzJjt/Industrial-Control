**DecToHex:**
Implement a self-contained function block in IEC 61131-3 structured text to convert a 10-digit decimal value to hexadecimal. Ensure proper handling of large values and edge cases.

**R-I-S-E:**

🟥 R (Role) – Define Your Role

Act as a PLC software developer implementing reliable and reusable conversion utilities in IEC 61131-3 Structured Text for industrial applications.

⸻

🟩 I (Input) – Information Provided

You need to develop a function block that:
	•	Accepts a 10-digit decimal input (e.g., up to 9999999999)
	•	Converts this value to a hexadecimal representation
	•	Handles large values and edge cases, such as overflow risk, zero input, or maximum data type limits
	•	Returns the result as a STRING suitable for display, logging, or transmission
	•	Must be self-contained and operate safely within scan-cycle execution

⸻

🟧 S (Steps) – Steps to Perform
	1.	Declare the function block with an input DecValue : LINT and an output HexString : STRING.
	2.	Initialize internal variables: remainder, digit character map (0 to F), a temporary string buffer, and a loop counter.
	3.	Implement the conversion logic using a loop:
	•	Divide the decimal value by 16
	•	Map the remainder to its corresponding hex digit
	•	Accumulate characters in reverse
	4.	Reverse the final result (if necessary) to correct the string order.
	5.	Add handling for special cases:
	•	Input = 0 → output "0"
	•	Input > max LINT → clip or raise a warning
	6.	Add inline comments explaining each major operation and safety consideration.

⸻

🟦 E (Expectation) – Expected Outcome

The completed function block should reliably convert any valid 10-digit decimal input into a hexadecimal string, with correct formatting and protection against invalid values. It will serve as a modular utility block for various PLC programs where hexadecimal display, data formatting, or communication protocol encoding is needed.
