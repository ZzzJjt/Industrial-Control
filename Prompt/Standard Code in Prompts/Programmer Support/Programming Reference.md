```
The IEC 61131-3 Structured Text (ST) programming language is used for specifying the behavior of programmable logic controllers (PLCs) and other industrial automation devices. Below is a list of keywords and constructs commonly used in ST, along with brief descriptions and examples where applicable.

Keywords and Constructs

Statements and Expressions

AND: Logical AND operator.
Iec61131


a AND b
OR: Logical OR operator.
Iec61131


a OR b
XOR: Logical XOR operator.
Iec61131


a XOR b
NOT: Logical NOT operator.
Iec61131


NOT a
EQ: Equality operator.
Iec61131


a EQ b
NE: Not equal operator.
Iec61131


a NE b
LT: Less than operator.
Iec61131


a LT b
LE: Less than or equal to operator.
Iec61131


a LE b
GT: Greater than operator.
Iec61131


a GT b
GE: Greater than or equal to operator.
Iec61131


a GE b
+: Addition operator.
Iec61131


a + b
-: Subtraction operator.
Iec61131


a - b
*: Multiplication operator.
Iec61131


a * b
/: Division operator.
Iec61131


a / b
DIV: Integer division operator.
Iec61131


a DIV b
MOD: Modulo operator.
Iec61131


a MOD b
^: Exponentiation operator.
Iec61131


a ^ b
Control Flow Structures

IF ... THEN ... ELSE ... END_IF: Conditional statement.
Iec61131


IF a > b THEN
    c := a;
ELSE
    c := b;
END_IF;
CASE ... OF ... END_CASE: Selection statement.
Iec61131


CASE a OF
    1: b := 10;
    2: b := 20;
    ELSE b := 0;
END_CASE;
WHILE ... DO ... END_WHILE: Loop statement.
Iec61131


WHILE a < 10 DO
    a := a + 1;
END_WHILE;
FOR ... TO ... DO ... END_FOR: Loop statement.
Iec61131


FOR i := 1 TO 10 DO
    a := a + 1;
END_FOR;
EXIT: Terminates the nearest enclosing loop or CASE statement.
Iec61131


WHILE True DO
    IF a > 10 THEN
        EXIT;
    END_IF;
END_WHILE;
CONTINUE: Skips the rest of the loop body and proceeds to the next iteration.
Iec61131


FOR i := 1 TO 10 DO
    IF i MOD 2 = 0 THEN
        CONTINUE;
    END_IF;
    a := a + 1;
END_FOR;
Data Types

BOOL: Boolean data type.
Iec61131


a := TRUE;
INT: Signed integer data type.
Iec61131


a := 10;
UINT: Unsigned integer data type.
Iec61131


a := 10U;
DINT: Double precision signed integer data type.
Iec61131


a := 10L;
UDINT: Double precision unsigned integer data type.
Iec61131


a := 10LU;
REAL: Floating point data type.
Iec61131


a := 10.0;
LREAL: Long double precision floating point data type.
Iec61131


a := 10.0L;
STRING: Character string data type.
Iec61131


a := "Hello";
WSTRING: Wide character string data type.
Iec61131


a := W"Hello";
DATE: Date data type.
Iec61131


a := DATE#20240101;
TIME: Time data type.
Iec61131


a := TIME#120000;
DATE_AND_TIME: Date and time data type.
Iec61131


a := DATE_AND_TIME#20240101T120000;
TIME_OF_DAY: Time of day data type.
Iec61131


a := TIME_OF_DAY#120000;
ARRAY: Array data type.
Iec61131


a[1] := 10;
STRUCT: User-defined structure data type.
Iec61131


TYPE Point : STRUCT
    x : INT;
    y : INT;
END_STRUCT
END_TYPE

VAR
    p : Point;
END_VAR

p.x := 10;
p.y := 20;
ENUM: Enumeration data type.
Iec61131


TYPE Color : ENUM
    Red : INT := 1;
    Green : INT := 2;
    Blue : INT := 3;
END_ENUM
END_TYPE

VAR
    c : Color;
END_VAR

c := Color#Red;
Functions and Procedures

FUNCTION ... END_FUNCTION: Function declaration.
Iec61131


FUNCTION Add(a : INT; b : INT) : INT
    Add := a + b;
END_FUNCTION
FUNCTION_BLOCK ... END_FUNCTION_BLOCK: Function block declaration.
Iec61131


FUNCTION_BLOCK Increment
    VAR_INPUT
        value : INT;
    END_VAR
    VAR_OUTPUT
        result : INT;
    END_VAR
    result := value + 1;
END_FUNCTION_BLOCK
PROCEDURE ... END_PROCEDURE: Procedure declaration.
Iec61131


PROCEDURE PrintHello
    WRITE("Hello");
END_PROCEDURE
RETURN: Return statement.
Iec61131


FUNCTION Add(a : INT; b : INT) : INT
    RETURN a + b;
END_FUNCTION
VAR: Local variable declaration.
Iec61131


VAR
    a : INT;
END_VAR
VAR_INPUT: Input variable declaration.
Iec61131


FUNCTION_BLOCK Increment
    VAR_INPUT
        value : INT;
    END_VAR
END_FUNCTION_BLOCK
VAR_OUTPUT: Output variable declaration.
Iec61131


FUNCTION_BLOCK Increment
    VAR_OUTPUT
        result : INT;
    END_VAR
END_FUNCTION_BLOCK
VAR_INOUT: In/out variable declaration.
Iec61131


FUNCTION_BLOCK Swap
    VAR_INOUT
        a, b : INT;
    END_VAR
    temp := a;
    a := b;
    b := temp;
END_FUNCTION_BLOCK
This list should cover the fundamental aspects of the IEC 61131-3 Structured Text programming language. Note that the actual implementation and syntax might vary slightly depending on the specific compiler or toolchain being used.
```
