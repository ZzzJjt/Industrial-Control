# IEC 61131-3 Structured Text Reference Guide

## Table of Contents
1. **Control Flow Structures**
2. **Data Types**
3. **Operators and Logical Constants**
4. **Standard Functions and Function Blocks**
5. **Code Snippets for Illustration**

---

## 1. Control Flow Structures

### IF-THEN-ELSE
**Description**: Conditional execution based on a boolean expression.
**Syntax**:
```st
IF condition THEN
    // statements if true
ELSIF another_condition THEN
    // statements if another_condition is true
ELSE
    // statements if false
END_IF;
```
**Example**:
```st
IF MotorRunning THEN
    Speed := 100;
ELSIF SpeedOverride THEN
    Speed := OverrideValue;
ELSE
    Speed := 0;
END_IF;
```

### CASE
**Description**: Multi-way conditional execution based on the value of an expression.
**Syntax**:
```st
CASE selector OF
    constant1: 
        // statements for constant1
    constant2..constantN:
        // statements for constant2 to constantN
    ELSE
        // default statements
END_CASE;
```
**Example**:
```st
CASE state OF
    1:
        UpdateTemperaturesAndPressures(rawMatPrepTemp, rawMatPrepPressure);
    2:
        UpdateTemperaturesAndPressures(reactionTemp, reactionPressure);
    3:
        UpdateTemperaturesAndPressures(coolingTemp, coolingPressure);
    ELSE
        state := 1; // Reset to initial state
END_CASE;
```

### FOR Loop
**Description**: Repeated execution of a block of code with a counter.
**Syntax**:
```st
FOR variable := start TO end BY step DO
    // statements
END_FOR;
```
**Example**:
```st
FOR i := 0 TO 9 BY 1 DO
    Sum := Sum + Array[i];
END_FOR;
```

### WHILE Loop
**Description**: Repeated execution as long as a condition is true.
**Syntax**:
```st
WHILE condition DO
    // statements
END_WHILE;
```
**Example**:
```st
WHILE SensorValue < Threshold DO
    Wait(T#1s);
END_WHILE;
```

### REPEAT-UNTIL Loop
**Description**: Repeated execution until a condition becomes true.
**Syntax**:
```st
REPEAT
    // statements
UNTIL condition END_REPEAT;
```
**Example**:
```st
REPEAT
    ReadSensor(SensorValue);
UNTIL SensorValue >= Threshold END_REPEAT;
```

### EXIT Statement
**Description**: Exit from a loop prematurely.
**Syntax**:
```st
EXIT;
```
**Example**:
```st
FOR i := 0 TO 9 BY 1 DO
    IF Array[i] = TargetValue THEN
        FoundIndex := i;
        EXIT;
    END_IF;
END_FOR;
```

## 2. Data Types

### Basic Data Types
- **BOOL**: Boolean values (`TRUE`, `FALSE`)
- **INT**: Signed integer (-32768 to 32767)
- **DINT**: Double signed integer (-2147483648 to 2147483647)
- **REAL**: Single precision floating point number
- **TIME**: Time duration (e.g., T#10s)
- **STRING**: Character string
- **ARRAY**: Ordered collection of elements of the same type
- **STRUCT**: User-defined aggregate data type

**Examples**:
```st
VAR
    Enable : BOOL := FALSE;
    Counter : INT := 0;
    Temperature : REAL := 25.0;
    Duration : TIME := T#5m;
    Message : STRING := 'Hello';
    Values : ARRAY[1..10] OF INT;
    Point : STRUCT(X : REAL; Y : REAL);
END_VAR
```

## 3. Operators and Logical Constants

### Arithmetic Operators
- **+**: Addition
- **-**: Subtraction
- ***: Multiplication
- **/**: Division
- **MOD**: Modulo (remainder)

**Examples**:
```st
Sum := A + B;
Difference := A - B;
Product := A * B;
Quotient := A / B;
Remainder := A MOD B;
```

### Relational Operators
- **=**: Equal to
- **<>**: Not equal to
- **<**: Less than
- **<=**: Less than or equal to
- **>**: Greater than
- **>=**: Greater than or equal to

**Examples**:
```st
IF A = B THEN
    Result := TRUE;
ELSIF A <> B THEN
    Result := FALSE;
END_IF;
```

### Logical Operators
- **AND**: Logical AND
- **OR**: Logical OR
- **NOT**: Logical NOT
- **XOR**: Logical XOR

**Examples**:
```st
Result := (A AND B) OR (C XOR D);
Negated := NOT Condition;
```

### Assignment Operator
- **:=**: Assigns a value to a variable

**Example**:
```st
Counter := Counter + 1;
```

## 4. Standard Functions and Function Blocks

### Timers
- **TON**: On-Delay Timer
- **TOF**: Off-Delay Timer
- **TP**: Pulse Timer

**Examples**:
```st
TON1(IN := StartButton, PT := T#10s);
IF TON1.Q THEN
    Output := TRUE;
END_IF;

TOF1(IN := StopButton, PT := T#5s);
IF TOF1.Q THEN
    Output := FALSE;
END_IF;

TP1(IN := Trigger, PT := T#1s);
IF TP1.Q THEN
    PulseCount := PulseCount + 1;
END_IF;
```

### Mathematical Functions
- **ABS**: Absolute value
- **SQRT**: Square root
- **POW**: Power
- **EXP**: Exponential
- **LN**: Natural logarithm
- **LOG10**: Base-10 logarithm

**Examples**:
```st
AbsoluteValue := ABS(-5);
SquareRoot := SQRT(25);
PowerValue := POW(Base, Exponent);
ExponentialValue := EXP(Value);
NaturalLog := LN(Number);
Base10Log := LOG10(Number);
```

### Conversion Functions
- **INT_TO_REAL**: Convert integer to real
- **REAL_TO_INT**: Convert real to integer

**Examples**:
```st
RealValue := INT_TO_REAL(IntValue);
IntValue := REAL_TO_INT(RealValue);
```

## 5. Code Snippets for Illustration

### IF-THEN-ELSE Example
```st
IF MotorRunning THEN
    Speed := 100;
ELSE
    Speed := 0;
END_IF;
```

### CASE Example
```st
CASE state OF
    1:
        UpdateTemperaturesAndPressures(rawMatPrepTemp, rawMatPrepPressure);
    2:
        UpdateTemperaturesAndPressures(reactionTemp, reactionPressure);
    3:
        UpdateTemperaturesAndPressures(coolingTemp, coolingPressure);
    ELSE
        state := 1; // Reset to initial state
END_CASE;
```

### FOR Loop Example
```st
FOR i := 0 TO 9 BY 1 DO
    Sum := Sum + Array[i];
END_FOR;
```

### WHILE Loop Example
```st
WHILE SensorValue < Threshold DO
    Wait(T#1s);
END_WHILE;
```

### REPEAT-UNTIL Loop Example
```st
REPEAT
    ReadSensor(SensorValue);
UNTIL SensorValue >= Threshold END_REPEAT;
```

### EXIT Statement Example
```st
FOR i := 0 TO 9 BY 1 DO
    IF Array[i] = TargetValue THEN
        FoundIndex := i;
        EXIT;
    END_IF;
END_FOR;
```

### TON Timer Example
```st
TON1(IN := StartButton, PT := T#10s);
IF TON1.Q THEN
    Output := TRUE;
END_IF;
```

### Mathematical Function Example
```st
AbsoluteValue := ABS(-5);
SquareRoot := SQRT(25);
PowerValue := POW(Base, Exponent);
ExponentialValue := EXP(Value);
NaturalLog := LN(Number);
Base10Log := LOG10(Number);
```

### Conversion Function Example
```st
RealValue := INT_TO_REAL(IntValue);
IntValue := REAL_TO_INT(RealValue);
```

---

This reference guide provides a comprehensive overview of the valid keywords, data types, operators, functions, and function blocks used in IEC 61131-3 Structured Text. It includes descriptions, syntax notes, and practical examples to aid developers in writing reliable and efficient PLC programs.

You can export this document in PDF, Markdown, or HTML formats for internal documentation or IDE integration. Here is the Markdown format for easy copying and exporting:

```markdown
# IEC 61131-3 Structured Text Reference Guide

## Table of Contents
1. **Control Flow Structures**
2. **Data Types**
3. **Operators and Logical Constants**
4. **Standard Functions and Function Blocks**
5. **Code Snippets for Illustration**

---

## 1. Control Flow Structures

### IF-THEN-ELSE
**Description**: Conditional execution based on a boolean expression.
**Syntax**:
```st
IF condition THEN
    // statements if true
ELSIF another_condition THEN
    // statements if another_condition is true
ELSE
    // statements if false
END_IF;
```
**Example**:
```st
IF MotorRunning THEN
    Speed := 100;
ELSIF SpeedOverride THEN
    Speed := OverrideValue;
ELSE
    Speed := 0;
END_IF;
```

### CASE
**Description**: Multi-way conditional execution based on the value of an expression.
**Syntax**:
```st
CASE selector OF
    constant1: 
        // statements for constant1
    constant2..constantN:
        // statements for constant2 to constantN
    ELSE
        // default statements
END_CASE;
```
**Example**:
```st
CASE state OF
    1:
        UpdateTemperaturesAndPressures(rawMatPrepTemp, rawMatPrepPressure);
    2:
        UpdateTemperaturesAndPressures(reactionTemp, reactionPressure);
    3:
        UpdateTemperaturesAndPressures(coolingTemp, coolingPressure);
    ELSE
        state := 1; // Reset to initial state
END_CASE;
```

### FOR Loop
**Description**: Repeated execution of a block of code with a counter.
**Syntax**:
```st
FOR variable := start TO end BY step DO
    // statements
END_FOR;
```
**Example**:
```st
FOR i := 0 TO 9 BY 1 DO
    Sum := Sum + Array[i];
END_FOR;
```

### WHILE Loop
**Description**: Repeated execution as long as a condition is true.
**Syntax**:
```st
WHILE condition DO
    // statements
END_WHILE;
```
**Example**:
```st
WHILE SensorValue < Threshold DO
    Wait(T#1s);
END_WHILE;
```

### REPEAT-UNTIL Loop
**Description**: Repeated execution until a condition becomes true.
**Syntax**:
```st
REPEAT
    // statements
UNTIL condition END_REPEAT;
```
**Example**:
```st
REPEAT
    ReadSensor(SensorValue);
UNTIL SensorValue >= Threshold END_REPEAT;
```

### EXIT Statement
**Description**: Exit from a loop prematurely.
**Syntax**:
```st
EXIT;
```
**Example**:
```st
FOR i := 0 TO 9 BY 1 DO
    IF Array[i] = TargetValue THEN
        FoundIndex := i;
        EXIT;
    END_IF;
END_FOR;
```

## 2. Data Types

### Basic Data Types
- **BOOL**: Boolean values (`TRUE`, `FALSE`)
- **INT**: Signed integer (-32768 to 32767)
- **DINT**: Double signed integer (-2147483648 to 2147483647)
- **REAL**: Single precision floating point number
- **TIME**: Time duration (e.g., T#10s)
- **STRING**: Character string
- **ARRAY**: Ordered collection of elements of the same type
- **STRUCT**: User-defined aggregate data type

**Examples**:
```st
VAR
    Enable : BOOL := FALSE;
    Counter : INT := 0;
    Temperature : REAL := 25.0;
    Duration : TIME := T#5m;
    Message : STRING := 'Hello';
    Values : ARRAY[1..10] OF INT;
    Point : STRUCT(X : REAL; Y : REAL);
END_VAR
```

## 3. Operators and Logical Constants

### Arithmetic Operators
- **+**: Addition
- **-**: Subtraction
- ***: Multiplication
- **/**: Division
- **MOD**: Modulo (remainder)

**Examples**:
```st
Sum := A + B;
Difference := A - B;
Product := A * B;
Quotient := A / B;
Remainder := A MOD B;
```

### Relational Operators
- **=**: Equal to
- **<>**: Not equal to
- **<**: Less than
- **<=**: Less than or equal to
- **>**: Greater than
- **>=**: Greater than or equal to

**Examples**:
```st
IF A = B THEN
    Result := TRUE;
ELSIF A <> B THEN
    Result := FALSE;
END_IF;
```

### Logical Operators
- **AND**: Logical AND
- **OR**: Logical OR
- **NOT**: Logical NOT
- **XOR**: Logical XOR

**Examples**:
```st
Result := (A AND B) OR (C XOR D);
Negated := NOT Condition;
```

### Assignment Operator
- **:=**: Assigns a value to a variable

**Example**:
```st
Counter := Counter + 1;
```

## 4. Standard Functions and Function Blocks

### Timers
- **TON**: On-Delay Timer
- **TOF**: Off-Delay Timer
- **TP**: Pulse Timer

**Examples**:
```st
TON1(IN := StartButton, PT := T#10s);
IF TON1.Q THEN
    Output := TRUE;
END_IF;

TOF1(IN := StopButton, PT := T#5s);
IF TOF1.Q THEN
    Output := FALSE;
END_IF;

TP1(IN := Trigger, PT := T#1s);
IF TP1.Q THEN
    PulseCount := PulseCount + 1;
END_IF;
```

### Mathematical Functions
- **ABS**: Absolute value
- **SQRT**: Square root
- **POW**: Power
- **EXP**: Exponential
- **LN**: Natural logarithm
- **LOG10**: Base-10 logarithm

**Examples**:
```st
AbsoluteValue := ABS(-5);
SquareRoot := SQRT(25);
PowerValue := POW(Base, Exponent);
ExponentialValue := EXP(Value);
NaturalLog := LN(Number);
Base10Log := LOG10(Number);
```

### Conversion Functions
- **INT_TO_REAL**: Convert integer to real
- **REAL_TO_INT**: Convert real to integer

**Examples**:
```st
RealValue := INT_TO_REAL(IntValue);
IntValue := REAL_TO_INT(RealValue);
```

## 5. Code Snippets for Illustration

### IF-THEN-ELSE Example
```st
IF MotorRunning THEN
    Speed := 100;
ELSE
    Speed := 0;
END_IF;
```

### CASE Example
```st
CASE state OF
    1:
        UpdateTemperaturesAndPressures(rawMatPrepTemp, rawMatPrepPressure);
    2:
        UpdateTemperaturesAndPressures(reactionTemp, reactionPressure);
    3:
        UpdateTemperaturesAndPressures(coolingTemp, coolingPressure);
    ELSE
        state := 1; // Reset to initial state
END_CASE;
```

### FOR Loop Example
```st
FOR i := 0 TO 9 BY 1 DO
    Sum := Sum + Array[i];
END_FOR;
```

### WHILE Loop Example
```st
WHILE SensorValue < Threshold DO
    Wait(T#1s);
END_WHILE;
```

### REPEAT-UNTIL Loop Example
```st
REPEAT
    ReadSensor(SensorValue);
UNTIL SensorValue >= Threshold END_REPEAT;
```

### EXIT Statement Example
```st
FOR i := 0 TO 9 BY 1 DO
    IF Array[i] = TargetValue THEN
        FoundIndex := i;
        EXIT;
    END_IF;
END_FOR;
```

### TON Timer Example
```st
TON1(IN := StartButton, PT := T#10s);
IF TON1.Q THEN
    Output := TRUE;
END_IF;
```

### Mathematical Function Example
```st
AbsoluteValue := ABS(-5);
SquareRoot := SQRT(25);
PowerValue := POW(Base, Exponent);
ExponentialValue := EXP(Value);
NaturalLog := LN(Number);
Base10Log := LOG10(Number);
```

### Conversion Function Example
```st
RealValue := INT_TO_REAL(IntValue);
IntValue := REAL_TO_INT(RealValue);
```

---
```

This Markdown content can be exported to PDF or HTML using tools like Pandoc or integrated into an IDE for quick reference.
