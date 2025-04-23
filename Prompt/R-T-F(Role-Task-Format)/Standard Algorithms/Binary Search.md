**Binary Search:**
Implement a self-contained function block in IEC 61131-3 structured text to perform a binary search on an array of 100 integers. Include input validation and efficient handling of search boundaries.

**R-T-F:**

🟥 R (Role) – Define Your Role

Act as a PLC developer implementing optimized algorithms in IEC 61131-3 Structured Text for industrial automation systems.

⸻

🟩 T (Task) – Define the Objective

Develop a self-contained function block in Structured Text that performs a binary search on an array of 100 sorted integers. The function block should:
	•	Accept a target value and an enable flag as inputs
	•	Include input validation, such as checking that the array is within bounds
	•	Use efficient logic with search boundaries (Low, High, Mid)
	•	Output the index of the found value, or -1 if not found, along with a Found boolean
	•	Be designed for modularity, clarity, and real-time PLC execution

⸻

🟧 F (Format) – Specify the Output Format

Provide:
	•	A fully structured IEC 61131-3 function block declaration
	•	Input and output variable definitions
	•	Main logic using WHILE or REPEAT loops
	•	Handling of edge cases (e.g., value not found, invalid index range)
	•	Inline comments explaining the binary search logic and decisions
	•	Optional notes on performance in scan-cycle-sensitive systems
