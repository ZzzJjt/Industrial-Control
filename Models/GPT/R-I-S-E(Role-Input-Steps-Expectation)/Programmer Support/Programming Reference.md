IEC 61131-3 Structured Text (ST) Reference Guide

⸻

🔹 1. Reserved Keywords (Categorized)

🔸 Control Flow Keywords
	•	IF, THEN, ELSE, ELSIF, END_IF
	•	CASE, OF, END_CASE
	•	FOR, TO, BY, DO, END_FOR
	•	WHILE, END_WHILE
	•	REPEAT, UNTIL, END_REPEAT
	•	EXIT, RETURN

🔸 Declaration Keywords
	•	VAR, VAR_INPUT, VAR_OUTPUT, VAR_IN_OUT, VAR_TEMP, END_VAR
	•	PROGRAM, FUNCTION, FUNCTION_BLOCK, METHOD, INTERFACE, EXTENDS, IMPLEMENTS, END_METHOD
	•	TYPE, END_TYPE
	•	STRUCT, END_STRUCT
	•	ARRAY, OF, RANGE

🔸 Operators and Constants
	•	Logical: AND, OR, NOT, XOR
	•	Comparison: =, <>, <, >, <=, >=
	•	Arithmetic: +, -, *, /, MOD, **
	•	Assignment: :=
	•	Constants: TRUE, FALSE

⸻

🔹 2. Standard Data Types

🔸 Boolean
	•	BOOL

🔸 Integer
	•	BYTE, WORD, DWORD, LWORD
	•	SINT, INT, DINT, LINT
	•	USINT, UINT, UDINT, ULINT

🔸 Floating Point
	•	REAL, LREAL

🔸 Character and String
	•	CHAR, WCHAR, STRING, WSTRING

🔸 Time and Date
	•	TIME, DATE, TIME_OF_DAY, DATE_AND_TIME

⸻

🔹 3. Standard Function Blocks

🔸 TON (On-Delay Timer)
	•	Inputs: IN: BOOL, PT: TIME
	•	Outputs: Q: BOOL, ET: TIME
	•	Usage:

TON1(IN := Start, PT := T#5S);
IF TON1.Q THEN
  Motor := TRUE;
END_IF

🔸 TOF (Off-Delay Timer)
	•	Similar to TON but delays deactivation

🔸 TP (Pulse Timer)
	•	Triggers a fixed-duration pulse when IN rises

🔸 CTU (Count Up)
	•	Inputs: CU: BOOL, RESET: BOOL, PV: INT
	•	Outputs: Q: BOOL, CV: INT
	•	Usage:

CTU1(CU := Sensor, RESET := Reset, PV := 10);
IF CTU1.Q THEN
  Alarm := TRUE;
END_IF


⸻

🔹 4. Example Snippets

🔸 IF-THEN-ELSE

IF Temp > 100.0 THEN
  Fan := TRUE;
ELSE
  Fan := FALSE;
END_IF

🔸 FOR Loop

FOR i := 1 TO 10 DO
  Sum := Sum + i;
END_FOR

🔸 CASE Statement

CASE Mode OF
  1: Motor := TRUE;
  2: Fan := TRUE;
  ELSE
    Alarm := TRUE;
END_CASE


⸻

🔹 5. Tips for Best Practices
	•	Always initialize variables explicitly.
	•	Avoid infinite loops (WHILE TRUE) in scan-based execution.
	•	Modularize code with FUNCTION_BLOCKs and METHODs.
	•	Use enumerated types (TYPE MyEnum : (State1, State2);) for states.
	•	Keep timer and counter names unique and consistent.

⸻

This guide can be printed, embedded in your IDE, or uploaded to your internal wiki for easy reference.
