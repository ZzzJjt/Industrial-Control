IEC 61131-3 Structured Text Programming Reference
This reference provides a consolidated guide to IEC 61131-3 Structured Text (ST), a high-level programming language for PLCs in industrial automation. It includes all reserved keywords, control structures, data types, operators, standard functions, and function blocks, with practical examples to aid developers. The document is organized for quick lookup and includes code snippets to demonstrate usage in real-world automation tasks.
1. Keyword Categories
Control Flow Keywords



Keyword
Description
Example



IF
Begins a conditional statement.
IF Temp > 100.0 THEN Alarm := TRUE; END_IF;


ELSIF
Specifies an alternative condition in an IF block.
ELSIF Temp > 80.0 THEN Warning := TRUE;


ELSE
Default case in an IF block.
ELSE Alarm := FALSE;


END_IF
Closes an IF statement.
See above.


CASE
Begins a switch-like selection based on a variable.
CASE State OF 1: Valve := TRUE; ELSE Valve := FALSE; END_CASE;


OF
Specifies case values in a CASE statement.
See above.


END_CASE
Closes a CASE statement.
See above.


FOR
Initiates a loop with a counter.
FOR i := 0 TO 9 DO Sum := Sum + i; END_FOR;


TO
Specifies the upper bound in a FOR loop.
See above.


BY
Defines the step size in a FOR loop.
FOR i := 0 TO 100 BY 2 DO ... END_FOR;


DO
Marks the start of a loop body.
See above.


END_FOR
Closes a FOR loop.
See above.


WHILE
Initiates a loop that continues while a condition is true.
WHILE Counter < 5 DO Counter := Counter + 1; END_WHILE;


END_WHILE
Closes a WHILE loop.
See above.


REPEAT
Initiates a loop that continues until a condition is true.
REPEAT Counter := Counter + 1; UNTIL Counter >= 5 END_REPEAT;


UNTIL
Specifies the termination condition for a REPEAT loop.
See above.


END_REPEAT
Closes a REPEAT loop.
See above.


EXIT
Exits a loop immediately.
IF Error THEN EXIT; END_IF;


RETURN
Exits a function or method, optionally returning a value.
RETURN Result;


Data Type Keywords



Keyword
Description
Example



BOOL
Boolean value (TRUE/FALSE).
VAR Alarm : BOOL; END_VAR


BYTE
8-bit unsigned integer (0 to 255).
VAR Data : BYTE := 16#FF; END_VAR


WORD
16-bit unsigned integer (0 to 65535).
VAR Count : WORD; END_VAR


DWORD
32-bit unsigned integer.
VAR LongCount : DWORD; END_VAR


LWORD
64-bit unsigned integer.
VAR VeryLong : LWORD; END_VAR


SINT
8-bit signed integer (-128 to 127).
VAR Temp : SINT := -10; END_VAR


INT
16-bit signed integer (-32768 to 32767).
VAR Counter : INT := 0; END_VAR


DINT
32-bit signed integer.
VAR Total : DINT; END_VAR


LINT
64-bit signed integer.
VAR BigNum : LINT; END_VAR


USINT
8-bit unsigned integer (0 to 255).
VAR SmallCount : USINT; END_VAR


UINT
16-bit unsigned integer (0 to 65535).
VAR Speed : UINT; END_VAR


UDINT
32-bit unsigned integer.
VAR LargeCount : UDINT; END_VAR


ULINT
64-bit unsigned integer.
VAR HugeCount : ULINT; END_VAR


REAL
32-bit floating-point number.
VAR Pressure : REAL := 3.14; END_VAR


LREAL
64-bit floating-point number.
VAR HighPrecision : LREAL; END_VAR


TIME
Duration or time span (e.g., T#5s).
VAR Delay : TIME := T#10s; END_VAR


DATE
Calendar date (e.g., D#2025-05-16).
VAR Today : DATE; END_VAR


TIME_OF_DAY
Time of day (e.g., TOD#11:52:00).
VAR StartTime : TIME_OF_DAY; END_VAR


DATE_AND_TIME
Combined date and time.
VAR Timestamp : DATE_AND_TIME; END_VAR


STRING
Character string (default length 80).
VAR Message : STRING := 'Error'; END_VAR


WSTRING
Wide-character string (Unicode).
VAR WideMsg : WSTRING; END_VAR


ARRAY
Multi-dimensional array of any data type.
VAR Data : ARRAY[0..9] OF INT; END_VAR


STRUCT
User-defined composite data type.
STRUCT Point; X : REAL; Y : REAL; END_STRUCT;


ENUM
User-defined enumerated type.
TYPE State : (Idle, Running, Stopped); END_TYPE


Operator Keywords



Keyword
Description
Example



AND
Logical AND operation.
IF A AND B THEN ... END_IF;


OR
Logical OR operation.
IF A OR B THEN ... END_IF;


XOR
Logical exclusive OR operation.
Result := A XOR B;


NOT
Logical negation.
IF NOT Alarm THEN ... END_IF;


MOD
Modulo (remainder) operation.
Remainder := 10 MOD 3;


+
Addition or string concatenation.
Sum := A + B;


–
Subtraction.
Diff := A – B;


*
Multiplication.
Product := A * B;


/
Division.
Quotient := A / B;


**
Exponentiation.
Power := A ** 2;


=
Equality comparison.
IF A = B THEN ... END_IF;


<>
Inequality comparison.
IF A <> B THEN ... END_IF;


<
Less than comparison.
IF A < B THEN ... END_IF;


>
Greater than comparison.
IF A > B THEN ... END_IF;


<=
Less than or equal to.
IF A <= B THEN ... END_IF;


>=
Greater than or equal to.
IF A >= B THEN ... END_IF;


System Constants



Keyword
Description
Example



TRUE
Boolean true value.
Alarm := TRUE;


FALSE
Boolean false value.
Alarm := FALSE;


NULL
Null pointer or unassigned reference.
VAR MyRef : REFERENCE TO INT := NULL; END_VAR


Declaration Keywords



Keyword
Description
Example



VAR
Declares local variables.
VAR Counter : INT; END_VAR


VAR_INPUT
Declares input parameters.
VAR_INPUT Speed : REAL; END_VAR


VAR_OUTPUT
Declares output parameters.
VAR_OUTPUT Result : BOOL; END_VAR


VAR_IN_OUT
Declares input/output parameters.
VAR_IN_OUT Data : INT; END_VAR


VAR_GLOBAL
Declares global variables.
VAR_GLOBAL MaxSpeed : REAL; END_VAR


VAR_EXTERNAL
Declares external variables.
VAR_EXTERNAL Sensor : REAL; END_VAR


VAR_TEMP
Declares temporary variables.
VAR_TEMP Temp : INT; END_VAR


CONSTANT
Declares constant values.
CONSTANT Max : INT := 100; END_CONSTANT


RETAIN
Persists variable values across power cycles.
VAR RETAIN LastCount : INT; END_VAR


NON_RETAIN
Explicitly non-persistent variables.
VAR NON_RETAIN Temp : INT; END_VAR


Program Organization Keywords



Keyword
Description
Example



PROGRAM
Defines a program.
PROGRAM MyProg ... END_PROGRAM


FUNCTION
Defines a function with a single return value.
FUNCTION Add : INT ... END_FUNCTION


FUNCTION_BLOCK
Defines a stateful function block.
FUNCTION_BLOCK Motor ... END_FUNCTION_BLOCK


METHOD
Defines a method within a function block.
METHOD Start : BOOL ... END_METHOD


INTERFACE
Defines an interface for polymorphism.
INTERFACE IActuator ... END_INTERFACE


EXTENDS
Specifies inheritance for function blocks.
FUNCTION_BLOCK Valve EXTENDS Actuator


IMPLEMENTS
Specifies interface implementation.
FUNCTION_BLOCK Valve IMPLEMENTS IActuator


PUBLIC
Public access for methods/variables.
METHOD PUBLIC Start : BOOL


PRIVATE
Private access for methods/variables.
METHOD PRIVATE Init : BOOL


PROTECTED
Protected access for inherited classes.
VAR PROTECTED Data : INT;


VIRTUAL
Marks a method as overridable.
METHOD VIRTUAL Start : BOOL


OVERRIDE
Overrides a virtual method.
METHOD OVERRIDE Start : BOOL


ABSTRACT
Defines an abstract function block.
FUNCTION_BLOCK ABSTRACT Base


FINAL
Prevents method overriding.
METHOD FINAL Stop : BOOL


2. Standard Functions and Function Blocks
Standard Functions



Function
Description
Example
Notes



ABS
Returns the absolute value of a number.
Result := ABS(-5.0);
Input: ANY_NUM. Output: Same type.


SQRT
Calculates the square root.
Root := SQRT(16.0);
Input: REAL or LREAL (≥ 0). Output: Same type.


LN
Calculates the natural logarithm.
Log := LN(2.718);
Input: REAL or LREAL (> 0). Output: Same type.


LOG
Calculates the base-10 logarithm.
Log10 := LOG(100.0);
Input: REAL or LREAL (> 0). Output: Same type.


EXP
Calculates e raised to the input power.
Value := EXP(1.0);
Input: REAL or LREAL. Output: Same type.


SIN
Calculates the sine of an angle (radians).
Sine := SIN(3.14159/2);
Input: REAL or LREAL. Output: Same type (-1.0 to 1.0).


COS
Calculates the cosine of an angle (radians).
Cosine := COS(0.0);
Input: REAL or LREAL. Output: Same type (-1.0 to 1.0).


TAN
Calculates the tangent of an angle (radians).
Tangent := TAN(0.7854);
Input: REAL or LREAL (avoid ±π/2). Output: Same type.


MIN
Returns the minimum of two or more values.
Smallest := MIN(10, 5);
Input: ANY_NUM. Output: Same type.


MAX
Returns the maximum of two or more values.
Largest := MAX(10, 5);
Input: ANY_NUM. Output: Same type.


LIMIT
Clamps a value between minimum and maximum bounds.
Clamped := LIMIT(0.0, Value, 100.0);
Inputs: MIN, IN, MAX (ANY_NUM). Output: Same type.


SEL
Selects between two values based on a boolean.
Output := SEL(Condition, 0, 1);
Inputs: G (BOOL), IN0, IN1 (ANY). Output: Same type as inputs.


MUX
Selects one of multiple inputs based on an index.
Selected := MUX(2, A, B, C, D);
Inputs: K (INT), multiple ANY inputs. Output: Same type as inputs.


Example: Using Standard Functions
VAR
    Temp : REAL := -10.5;
    AbsTemp : REAL;
    Root : REAL;
    MaxVal : REAL;
END_VAR
AbsTemp := ABS(Temp);       (* AbsTemp = 10.5 *)
Root := SQRT(16.0);         (* Root = 4.0 *)
MaxVal := MAX(Temp, 0.0);   (* MaxVal = 0.0 *)

Standard Function Blocks



Function Block
Description
Example
Notes



TON
Timer On-Delay: Delays output activation.
Timer(IN := Start, PT := T#5s);
Inputs: IN (BOOL), PT (TIME). Outputs: Q (BOOL), ET (TIME).


TOF
Timer Off-Delay: Delays output deactivation.
Timer(IN := Stop, PT := T#3s);
Same as TON.


TP
Timer Pulse: Generates a pulse of specified duration.
Timer(IN := Trigger, PT := T#1s);
Same as TON.


CTU
Up Counter: Increments on rising edge.
Counter(CU := Pulse, R := Reset, PV := 10);
Inputs: CU (BOOL), R (BOOL), PV (INT). Outputs: Q (BOOL), CV (INT).


CTD
Down Counter: Decrements on rising edge.
Counter(CD := Pulse, LD := Load, PV := 10);
Inputs: CD (BOOL), LD (BOOL), PV (INT). Outputs: Q (BOOL), CV (INT).


CTUD
Up/Down Counter: Increments or decrements.
Counter(CU := Up, CD := Down, PV := 10);
Inputs: CU, CD, R, LD (BOOL), PV (INT). Outputs: QU, QD (BOOL), CV (INT).


R_TRIG
Rising Edge Trigger: Detects rising edge.
Trigger(CLK := Signal);
Input: CLK (BOOL). Output: Q (BOOL).


F_TRIG
Falling Edge Trigger: Detects falling edge.
Trigger(CLK := Signal);
Input: CLK (BOOL). Output: Q (BOOL).


Example: Using TON Timer
VAR
    Start : BOOL;
    Timer : TON;
    Output : BOOL;
END_VAR
Timer(IN := Start, PT := T#5s);
Output := Timer.Q;           (* TRUE after 5 seconds *)

Example: Using CTU Counter
VAR
    Pulse : BOOL;
    Reset : BOOL;
    Counter : CTU;
    CountReached : BOOL;
END_VAR
Counter(CU := Pulse, R := Reset, PV := 10);
CountReached := Counter.Q;   (* TRUE when count reaches 10 *)

3. Practical Usage Examples
Control Flow: IF and CASE
VAR
    Temp : REAL := 85.0;
    State : INT := 1;
    Alarm : BOOL;
    Valve : BOOL;
END_VAR
(* IF Example *)
IF Temp > 100.0 THEN
    Alarm := TRUE;
ELSIF Temp > 80.0 THEN
    Alarm := FALSE;
ELSE
    Alarm := FALSE;
END_IF;
(* CASE Example *)
CASE State OF
    1: Valve := TRUE;
    2: Valve := FALSE;
ELSE
    Valve := FALSE;
END_CASE;

Loop: FOR
VAR
    i : INT;
    Sum : INT := 0;
    Data : ARRAY[0..4] OF INT := [1, 2, 3, 4, 5];
END_VAR
FOR i := 0 TO 4 DO
    Sum := Sum + Data[i];
END_FOR; (* Sum = 15 *)

Data Type: STRUCT and ARRAY
TYPE Point : STRUCT
    X : REAL;
    Y : REAL;
END_STRUCT;
END_TYPE
VAR
    Position : Point;
    Values : ARRAY[0..2] OF REAL := [1.0, 2.0, 3.0];
END_VAR
Position.X := 10.0;
Position.Y := Values[1]; (* Y = 2.0 *)

Function Block: R_TRIG with TON
VAR
    Button : BOOL;
    Trigger : R_TRIG;
    Timer : TON;
    Output : BOOL;
END_VAR
Trigger(CLK := Button);         (* Detect rising edge *)
Timer(IN := Trigger.Q, PT := T#3s);
Output := Timer.Q;              (* TRUE 3s after button press *)

4. Notes for Developers

Vendor-Specific Behavior: Some platforms (e.g., CODESYS, TwinCAT) extend standard functions or add proprietary ones. Always check the PLC’s documentation for deviations (e.g., additional string functions).
Execution Context: Structured Text runs in cyclic PLC scans (e.g., 10–100 ms). Avoid heavy computations in FOR or WHILE loops to prevent scan cycle overruns.
Portability: Stick to standard keywords and functions for cross-platform compatibility. Vendor-specific extensions may not work on all IEC 61131-3 PLCs.
Debugging: Use HMI or PLC debugging tools to monitor variables (e.g., Timer.ET, Counter.CV) and test control flow.
Resources: Refer to the IEC 61131-3 standard, CODESYS documentation, or PLCopen guidelines for detailed specifications.

This reference consolidates IEC 61131-3 Structured Text constructs, providing a quick lookup for control engineers and developers. Use the examples to build robust automation programs, and validate against your PLC platform for optimal performance.
