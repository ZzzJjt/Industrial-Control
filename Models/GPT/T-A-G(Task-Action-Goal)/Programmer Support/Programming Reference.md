📂 1. Data Types

🔹 Basic Types
Keyword
Description
Example
BOOL
Boolean (TRUE/FALSE)
VAR X : BOOL := TRUE;
INT
16-bit signed integer
VAR Speed : INT := 100;
DINT
32-bit signed integer
VAR Count : DINT;
REAL
32-bit float
VAR Temp : REAL := 22.5;
LREAL
64-bit float
VAR Precision : LREAL;
TIME
Time duration
VAR Delay : TIME := T#5s;
STRING
Character string

VAR SensorArray : ARRAY[1..4] OF REAL;

📂 2. Control Flow Keywords

🔹 Conditional

Keyword
Use
Example
IF, THEN, ELSIF, ELSE, END_IF
Conditional logic

📂 3. Function and Function Block Calls

🔹 Timers and Counters
Block
Description
Example
TON
On-delay timer
TOF
Off-delay timer
TP
Pulse timer
CTU
Count up
CTD
Count down
CTUD
Count up/down

TON1(IN := Start, PT := T#5s);
IF TON1.Q THEN
    Valve := TRUE;
END_IF;
