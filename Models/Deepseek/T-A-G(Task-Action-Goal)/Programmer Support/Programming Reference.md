
---

# IEC 61131-3 Structured Text (ST) Programming Reference Guide

## Table of Contents
1. **Control Flow Structures**
2. **Standard Functions**
3. **Data Types**
4. **Operators**
5. **Reserved Words**
6. **Special Instructions**

## 1. Control Flow Structures

### IF...ELSIF...ELSE...END_IF
**Description**: Conditional execution based on boolean expressions.
**Usage Example**:
```st
VAR
    Temperature : REAL := 250.0;
    Status : STRING;
END_VAR

IF Temperature > 300 THEN
    Status := 'High';
ELSIF Temperature < 200 THEN
    Status := 'Low';
ELSE
    Status := 'Normal';
END_IF;
```

### CASE...OF...END_CASE
**Description**: Multi-way conditional selection.
**Usage Example**:
```st
VAR
    State : INT := 1;
    Action : STRING;
END_VAR

CASE State OF
    1:
        Action := 'Initialize';
    2:
        Action := 'Process';
    3:
        Action := 'Finalize';
    ELSE
        Action := 'Unknown';
END_CASE;
```

### FOR...DO...END_FOR
**Description**: Loop with a counter variable.
**Usage Example**:
```st
VAR
    i : INT;
    Sum : INT := 0;
END_VAR

FOR i := 1 TO 10 DO
    Sum := Sum + i;
END_FOR;
```

### WHILE...DO...END_WHILE
**Description**: Loop until a condition becomes false.
**Usage Example**:
```st
VAR
    Counter : INT := 0;
END_VAR

WHILE Counter < 5 DO
    Counter := Counter + 1;
END_WHILE;
```

### REPEAT...UNTIL
**Description**: Loop until a condition becomes true.
**Usage Example**:
```st
VAR
    Counter : INT := 0;
END_VAR

REPEAT
    Counter := Counter + 1;
UNTIL Counter >= 5;
```

## 2. Standard Functions

### TON (On-Delay Timer)
**Description**: On-delay timer.
**Usage Example**:
```st
VAR
    MyTimer : TON;
END_VAR

MyTimer(IN := TRUE, PT := T#5s);
IF MyTimer.Q THEN
    // Timer has expired
END_IF;
```

### CTU (Count-Up Counter)
**Description**: Count-up counter.
**Usage Example**:
```st
VAR
    MyCounter : CTU;
END_VAR

MyCounter(CU := TRUE, R := FALSE, PV := 10);
IF MyCounter.Q THEN
    // Counter has reached preset value
END_IF;
```

### ABS (Absolute Value)
**Description**: Returns the absolute value of a number.
**Usage Example**:
```st
VAR
    Value : REAL := -10.5;
    AbsValue : REAL;
END_VAR

AbsValue := ABS(Value); // AbsValue will be 10.5
```

### SIN (Sine Function)
**Description**: Computes the sine of an angle in radians.
**Usage Example**:
```st
VAR
    Angle : REAL := PI / 2; // 90 degrees in radians
    SineValue : REAL;
END_VAR

SineValue := SIN(Angle); // SineValue will be 1.0
```

### COS (Cosine Function)
**Description**: Computes the cosine of an angle in radians.
**Usage Example**:
```st
VAR
    Angle : REAL := 0.0; // 0 degrees in radians
    CosineValue : REAL;
END_VAR

CosineValue := COS(Angle); // CosineValue will be 1.0
```

### TAN (Tangent Function)
**Description**: Computes the tangent of an angle in radians.
**Usage Example**:
```st
VAR
    Angle : REAL := PI / 4; // 45 degrees in radians
    TangentValue : REAL;
END_VAR

TangentValue := TAN(Angle); // TangentValue will be 1.0
```

### ASIN (Arc Sine Function)
**Description**: Computes the arc sine of a value.
**Usage Example**:
```st
VAR
    Value : REAL := 1.0;
    ArcSineValue : REAL;
END_VAR

ArcSineValue := ASIN(Value); // ArcSineValue will be PI / 2 (90 degrees in radians)
```

### ACOS (Arc Cosine Function)
**Description**: Computes the arc cosine of a value.
**Usage Example**:
```st
VAR
    Value : REAL := 1.0;
    ArcCosineValue : REAL;
END_VAR

ArcCosineValue := ACOS(Value); // ArcCosineValue will be 0.0 (0 degrees in radians)
```

### ATAN (Arc Tangent Function)
**Description**: Computes the arc tangent of a value.
**Usage Example**:
```st
VAR
    Value : REAL := 1.0;
    ArcTangentValue : REAL;
END_VAR

ArcTangentValue := ATAN(Value); // ArcTangentValue will be PI / 4 (45 degrees in radians)
```

### ATAN2 (Arc Tangent of Two Values)
**Description**: Computes the arc tangent of y/x using the signs of arguments to determine the correct quadrant.
**Usage Example**:
```st
VAR
    Y : REAL := 1.0;
    X : REAL := 1.0;
    ArcTangent2Value : REAL;
END_VAR

ArcTangent2Value := ATAN2(Y, X); // ArcTangent2Value will be PI / 4 (45 degrees in radians)
```

### EXP (Exponential Function)
**Description**: Computes the exponential of a number (e^x).
**Usage Example**:
```st
VAR
    Exponent : REAL := 1.0;
    ExpValue : REAL;
END_VAR

ExpValue := EXP(Exponent); // ExpValue will be approximately 2.71828
```

### LN (Natural Logarithm)
**Description**: Computes the natural logarithm (base e) of a number.
**Usage Example**:
```st
VAR
    Value : REAL := 2.71828;
    LnValue : REAL;
END_VAR

LnValue := LN(Value); // LnValue will be approximately 1.0
```

### LOG10 (Base 10 Logarithm)
**Description**: Computes the base 10 logarithm of a number.
**Usage Example**:
```st
VAR
    Value : REAL := 100.0;
    Log10Value : REAL;
END_VAR

Log10Value := LOG10(Value); // Log10Value will be 2.0
```

### SQRT (Square Root)
**Description**: Computes the square root of a number.
**Usage Example**:
```st
VAR
    Value : REAL := 16.0;
    SqrtValue : REAL;
END_VAR

SqrtValue := SQRT(Value); // SqrtValue will be 4.0
```

## 3. Data Types

### Basic Data Types
- **BOOL**: Boolean (TRUE/FALSE)
- **BYTE**: Unsigned 8-bit integer (0 to 255)
- **WORD**: Unsigned 16-bit integer (0 to 65535)
- **DWORD**: Unsigned 32-bit integer (0 to 4294967295)
- **LWORD**: Unsigned 64-bit integer (0 to 18446744073709551615)
- **SINT**: Signed 8-bit integer (-128 to 127)
- **INT**: Signed 16-bit integer (-32768 to 32767)
- **DINT**: Signed 32-bit integer (-2147483648 to 2147483647)
- **LINT**: Signed 64-bit integer (-9223372036854775808 to 9223372036854775807)
- **USINT**: Unsigned 8-bit integer (0 to 255)
- **UINT**: Unsigned 16-bit integer (0 to 65535)
- **UDINT**: Unsigned 32-bit integer (0 to 4294967295)
- **ULINT**: Unsigned 64-bit integer (0 to 18446744073709551615)
- **REAL**: 32-bit floating-point number
- **LREAL**: 64-bit floating-point number
- **TIME**: Time duration
- **DATE**: Date
- **DT**: Date and time
- **TOD**: Time of day
- **STRING**: Variable-length string

### Derived Data Types
- **ARRAY**: Ordered collection of elements of the same type.
- **STRUCT**: User-defined aggregate data type containing named fields.
- **UNION**: Overlapping memory region for different data types.

**Example of ARRAY**:
```st
VAR
    Temperatures : ARRAY[1..10] OF REAL;
END_VAR
```

**Example of STRUCT**:
```st
TYPE SensorData :
STRUCT
    Temperature : REAL;
    Pressure : REAL;
END_STRUCT
END_TYPE

VAR
    MySensor : SensorData;
END_VAR
```

## 4. Operators

### Arithmetic Operators
- **+**: Addition
- **-**: Subtraction
- ***: Multiplication
- **/**: Division
- **MOD**: Modulo (remainder)

**Usage Example**:
```st
VAR
    A : INT := 10;
    B : INT := 3;
    Result : INT;
END_VAR

Result := A + B;  // Result = 13
Result := A - B;  // Result = 7
Result := A * B;  // Result = 30
Result := A / B;  // Result = 3
Result := A MOD B; // Result = 1
```

### Relational Operators
- **=**: Equal to
- **<>**: Not equal to
- **<**: Less than
- **<=**: Less than or equal to
- **>**: Greater than
- **>=**: Greater than or equal to

**Usage Example**:
```st
VAR
    A : INT := 10;
    B : INT := 5;
    Condition : BOOL;
END_VAR

Condition := A = B;  // Condition = FALSE
Condition := A <> B; // Condition = TRUE
Condition := A < B;  // Condition = FALSE
Condition := A <= B; // Condition = FALSE
Condition := A > B;  // Condition = TRUE
Condition := A >= B; // Condition = TRUE
```

### Logical Operators
- **AND**: Logical AND
- **OR**: Logical OR
- **NOT**: Logical NOT
- **XOR**: Exclusive OR

**Usage Example**:
```st
VAR
    A : BOOL := TRUE;
    B : BOOL := FALSE;
    Result : BOOL;
END_VAR

Result := A AND B; // Result = FALSE
Result := A OR B;  // Result = TRUE
Result := NOT A;   // Result = FALSE
Result := A XOR B; // Result = TRUE
```

### Bitwise Operators
- **AND**: Bitwise AND
- **OR**: Bitwise OR
- **XOR**: Bitwise XOR
- **NOT**: Bitwise NOT
- **SHL**: Shift left
- **SHR**: Shift right

**Usage Example**:
```st
VAR
    A : INT := 10; // Binary: 1010
    B : INT := 3;  // Binary: 0011
    Result : INT;
END_VAR

Result := A AND B; // Result = 2 (Binary: 0010)
Result := A OR B;  // Result = 11 (Binary: 1011)
Result := A XOR B; // Result = 9 (Binary: 1001)
Result := NOT A;   // Result = -11 (Binary: ...1111111111110101)
Result := A SHL 1; // Result = 20 (Binary: 10100)
Result := A SHR 1; // Result = 5 (Binary: 0101)
```

## 5. Reserved Words

Here is a list of reserved words in IEC 61131-3 ST that cannot be used as identifiers:

```
ABS, AND, ARRAY, BEGIN, BYTEDATA, CASE, CONSTANT, DATE, DATEDATA, DT, DT_DATA, DUTYCYCLE,
ELSE, ELSIF, END_ARRAY, END_CASE, END_FOR, END_FUNCTION, END_FUNCTION_BLOCK, END_IF,
END_METHOD, END_PROGRAM, END_REPEAT, END_STRUCTURE, END_TYPE, END_VAR, END_WHILE, EXIT,
FALSE, FOR, FUNCTION, FUNCTION_BLOCK, GE, GT, HIGH, IF, IMPLEMENTATION, IMPORT, IN, INITIAL_STEP,
INPUT, INPUT_IN_OUT, INSTANCEOF, INTO, INTERLOCKED, LATCH, LD, LEFT, LEN, LESS_THAN_OR_EQUAL,
LESS_THAN, LIMIT, LIST, LOAD, LOCATION, LPAREN, LT, MEMBERSHIP, METHOD, MORE_THAN_OR_EQUAL,
MORE_THAN, NETWORK, NEQ, NO_ERROR, NOT, NULL, OF, ON, OR, OUTPUT, PARAMETER, PERSISTENT,
PRAGMA, PROGRAM, PROTECTED, PT, Q, R, RANGE, RECEIVE, RECTIFY, REF_TO, REPEAT, RETURN, RIGHT,
RUNNING, SCONST, SELECT, SET, SIZEOF, SKIP, STEP, STOP, STRUCTURE, SUBPROGRAM, TEMP, THEN,
TO, TIME, TIME_OF_DAY, TIMEDIFF, TIMEOUT, TIMEREF, TRANSITION, TRUE, TYPE, ULINT, UNTIL,
USE, VAR, VAR_CONFIG, VAR_EXTERNAL, VAR_GLOBAL, VAR_INPUT, VAR_IN_OUT, VAR_LOCAL, VAR_OUTPUT,
VAR_TEMP, WAIT, WSTRING, WITH, XOR, YES
```

## 6. Special Instructions

### REF_TO
**Description**: Creates a reference to a variable.
**Usage Example**:
```st
VAR
    MyVar : INT := 10;
    RefToMyVar : REF_TO INT;
END_VAR

RefToMyVar := ADR(MyVar);
IF RefToMyVar <> NULL THEN
    RefToMyVar^ := 20; // MyVar is now 20
END_IF;
```

### ADR
**Description**: Gets the address of a variable.
**Usage Example**:
```st
VAR
    MyVar : INT := 10;
    Address : REF_TO INT;
END_VAR

Address := ADR(MyVar);
IF Address <> NULL THEN
    Address^ := 20; // MyVar is now 20
END_IF;
```

### ^ (Pointer Dereference)
**Description**: Dereferences a pointer to access the variable it points to.
**Usage Example**:
```st
VAR
    MyVar : INT := 10;
    RefToMyVar : REF_TO INT;
END_VAR

RefToMyVar := ADR(MyVar);
IF RefToMyVar <> NULL THEN
    RefToMyVar^ := 20; // MyVar is now 20
END_IF;
```

### NULL
**Description**: Represents a null reference.
**Usage Example**:
```st
VAR
    RefToMyVar : REF_TO INT;
END_VAR

RefToMyVar := NULL;
IF RefToMyVar = NULL THEN
    // Handle null reference
END_IF;
```

### SIZEOF
**Description**: Returns the size of a variable or data type.
**Usage Example**:
```st
VAR
    MyInt : INT;
    Size : INT;
END_VAR

Size := SIZEOF(MyInt); // Size will be 2 bytes
```

### TO_STRING
**Description**: Converts a numeric value to a string.
**Usage Example**:
```st
VAR
    Number : INT := 123;
    Str : STRING;
END_VAR

Str := TO_STRING(Number); // Str will be "123"
```

### TO_INT
**Description**: Converts a string to an integer.
**Usage Example**:
```st
VAR
    Str : STRING := "123";
    Number : INT;
END_VAR

Number := TO_INT(Str); // Number will be 123
```

### TO_REAL
**Description**: Converts a string to a real number.
**Usage Example**:
```st
VAR
    Str : STRING := "123.45";
    Number : REAL;
END_VAR

Number := TO_REAL(Str); // Number will be 123.45
```

## Conclusion

This comprehensive reference guide provides a detailed overview of the IEC 61131-3 Structured Text (ST) language, including control flow structures, standard functions, data types, operators, reserved words, and special instructions. It serves as a valuable resource for engineers and developers to write correct, efficient, and standards-compliant ST code for industrial control systems.

Feel free to customize or expand this guide based on specific project needs or additional features found in the latest version of IEC 61131-3.
