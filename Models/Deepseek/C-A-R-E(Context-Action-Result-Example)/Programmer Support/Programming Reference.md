Certainly! Below is a comprehensive, categorized reference document for IEC 61131-3 Structured Text (ST). This document includes all reserved keywords, core data types, logical and arithmetic operators, system constants and standard function blocks, along with practical usage examples.

### Comprehensive Reference Document for IEC 61131-3 Structured Text

#### Keywords

**Control Structures:**
- `IF`, `THEN`, `ELSE`, `ELSIF`, `END_IF`
- `CASE`, `OF`, `END_CASE`
- `FOR`, `TO`, `BY`, `DO`, `END_FOR`
- `WHILE`, `REPEAT`, `UNTIL`, `END_REPEAT`
- `RETURN`
- `EXIT`, `CONTINUE`

**Variable Declaration:**
- `VAR`, `END_VAR`
- `VAR_INPUT`, `END_VAR`
- `VAR_OUTPUT`, `END_VAR`
- `VAR_IN_OUT`, `END_VAR`
- `CONSTANT`
- `RETAIN`

**Data Types:**
- `BOOL`
- `INT`, `DINT`, `LINT`
- `UINT`, `UDINT`, `ULINT`
- `REAL`, `LREAL`
- `TIME`, `DT`, `DATE_AND_TIME`, `TIME_OF_DAY`, `DATE`
- `STRING`
- `ARRAY`
- `REF_TO`
- `ANY`
- `ANY_DERIVED`
- `ANY_ELEMENTARY`
- `ANY_MAGNITUDE`
- `ANY_NUM`
- `ANY_REAL`
- `ANY_INT`
- `ANY_BIT`
- `ANY_STRING`
- `ANY_DATE`
- `ANY_DURATION`

**Logical Operators:**
- `AND`
- `OR`
- `NOT`
- `XOR`
- `:=` (assignment)
- `=` (equality)
- `<>` (inequality)
- `<` (less than)
- `>` (greater than)
- `<=` (less than or equal to)
- `>=` (greater than or equal to)

**Arithmetic Operators:**
- `+` (addition)
- `-` (subtraction)
- `*` (multiplication)
- `/` (division)
- `MOD` (modulus)
- `DIV` (integer division)

**System Constants:**
- `TRUE`
- `FALSE`
- `NULL`
- `NONE`
- `UNKNOWN`
- `DERIVED`
- `ELEMENTARY`
- `MAGNITUDE`
- `NUM`
- `REAL`
- `INT`
- `BIT`
- `STRING`
- `DATE`
- `DURATION`

#### Standard Function Blocks

**Timers:**
- `TON` (On-Delay Timer)
- `TOF` (Off-Delay Timer)
- `TP` (Pulse Timer)

**Counters:**
- `CTU` (Count-Up Counter)
- `CTD` (Count-Down Counter)
- `CTUD` (Count-Up/Down Counter)

**Mathematical Functions:**
- `ABS(x)` - Absolute value of `x`
- `SIN(x)` - Sine of `x`
- `COS(x)` - Cosine of `x`
- `TAN(x)` - Tangent of `x`
- `ASIN(x)` - Arcsine of `x`
- `ACOS(x)` - Arccosine of `x`
- `ATAN(x)` - Arctangent of `x`
- `ATAN2(y, x)` - Four-quadrant arctangent of `y/x`
- `EXP(x)` - Exponential of `x` (e^x)
- `LN(x)` - Natural logarithm of `x`
- `LOG10(x)` - Base-10 logarithm of `x`
- `POW(x, n)` - Power of `x` raised to `n`
- `SQRT(x)` - Square root of `x`

**Conversion Functions:**
- `FLOAT_TO_INT(value)` - Convert float to integer
- `INT_TO_FLOAT(value)` - Convert integer to float
- `DEG_TO_RAD(angle_deg)` - Convert degrees to radians
- `RAD_TO_DEG(angle_rad)` - Convert radians to degrees

#### Code Examples

##### IF-THEN-ELSE Syntax

```st
VAR
    Temp : REAL := 85.0;
    Overheat : BOOL;
END_VAR

IF Temp > 100.0 THEN
    Overheat := TRUE;
ELSE
    Overheat := FALSE;
END_IF;
```

##### CASE Statement

```st
VAR
    state : INT := 1;
    action : STRING;
END_VAR

CASE state OF
    1:
        action := 'Start';
    2:
        action := 'Run';
    3:
        action := 'Stop';
    ELSE
        action := 'Unknown';
END_CASE;
```

##### FOR Loop

```st
VAR
    i : INT;
    sum : INT := 0;
END_VAR

FOR i := 1 TO 10 DO
    sum := sum + i;
END_FOR;
```

##### WHILE Loop

```st
VAR
    count : INT := 0;
END_VAR

WHILE count < 10 DO
    count := count + 1;
END_WHILE;
```

##### REPEAT-UNTIL Loop

```st
VAR
    count : INT := 0;
END_VAR

REPEAT
    count := count + 1;
UNTIL count >= 10 END_REPEAT;
```

##### TON (On-Delay Timer) Function Block

```st
VAR
    Timer1 : TON;
    Start : BOOL;
END_VAR

Timer1(IN := Start, PT := T#5s);
IF Timer1.Q THEN
    // Execute action when timer completes
END_IF;
```

##### Array Declaration and Usage

```st
VAR
    temperatures : ARRAY[1..5] OF REAL := [20.0, 25.0, 30.0, 35.0, 40.0];
    avg_temp : REAL;
    sum : REAL := 0.0;
    i : INT;
END_VAR

FOR i := 1 TO 5 DO
    sum := sum + temperatures[i];
END_FOR;

avg_temp := sum / 5.0;
```

##### CTU (Count-Up Counter) Function Block

```st
VAR
    Counter1 : CTU;
    CountUp : BOOL;
    Reset : BOOL;
END_VAR

Counter1(CU := CountUp, R := Reset, PV => current_count);
```

##### Mathematical Functions

```st
VAR
    x : REAL := 2.718;
    y : REAL := 1.57;
    result : REAL;
END_VAR

result := EXP(x); // e^2.718
result := SIN(y); // sin(1.57)
result := SQRT(result); // sqrt(sin(1.57))
```

### Conclusion

This comprehensive reference document serves as a clear and structured guide for IEC 61131-3 Structured Text. It includes all necessary keywords, data types, operators, system constants, and standard function blocks, along with practical usage examples. This document will help developers write compliant and efficient code, serve as a daily reference tool, and reduce syntax errors and debugging time.

### Recommended Formatting Options

1. **PDF Cheat Sheet**: Ideal for quick lookup and offline access.
2. **Webpage**: Accessible online for easy updates and sharing.
3. **IDE-Integrated Tooltip System**: Provides instant information directly within the development environment.

Feel free to format this document according to your preferred medium for optimal use.
