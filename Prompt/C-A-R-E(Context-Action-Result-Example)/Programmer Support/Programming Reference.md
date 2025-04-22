**Programming Reference:**
Compile a comprehensive list of all valid keywords used in the IEC 61131-3 Structured Text programming language, ensuring that it covers both standard control flow structures and data types. Include examples of usage for key functions where applicable.

**C-A-R-E:**

🟥 C (Context)

IEC 61131-3 Structured Text is a widely used programming language in industrial automation. Despite its standardized nature, many developers lack a single, comprehensive reference that clearly lists all valid keywords, data types, and control structures, along with practical usage examples. This makes learning, troubleshooting, and implementing efficient code more challenging—especially for new programmers or teams working across different vendors.

⸻

🟩 A (Action)

Create a comprehensive, categorized reference document that includes:
	•	All reserved keywords in Structured Text (ST), such as control structures (IF, CASE, FOR, WHILE, etc.)
	•	Core data types (BOOL, INT, REAL, TIME, STRING, ARRAY, etc.)
	•	Logical and arithmetic operators (AND, OR, MOD, NOT, +, -, etc.)
	•	System constants and standard function blocks (TRUE, FALSE, TON, CTU, ABS, SQRT, etc.)
	•	Code examples for commonly used constructs (e.g., how to use a timer, write a loop, or declare arrays)

⸻

🟨 R (Result)

The result will be a clear, structured, and practical programming reference for IEC 61131-3 Structured Text, which:
	•	Makes it faster and easier for developers to write compliant and efficient code
	•	Serves as a daily reference tool for both new and experienced automation engineers
	•	Helps reduce syntax errors, debugging time, and onboarding effort
	•	Can be formatted as a PDF cheat sheet, a webpage, or even an IDE-integrated tooltip system

⸻

🟦 E (Example)

IF-THEN-ELSE syntax:

VAR
  Temp : REAL := 85.0;
  Overheat : BOOL;
END_VAR

IF Temp > 100.0 THEN
  Overheat := TRUE;
ELSE
  Overheat := FALSE;
END_IF;

TON (On-delay timer) function block:

VAR
  Timer1 : TON;
  Start  : BOOL;
END_VAR

Timer1(IN := Start, PT := T#5s);
IF Timer1.Q THEN
  // Execute action when timer completes
END_IF;
