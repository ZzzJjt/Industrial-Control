# 📘 **IEC 61131-3 Structured Text (ST) Programming Reference**

This document provides a **comprehensive reference guide** for IEC 61131-3 Structured Text, designed to help PLC developers write clean, reliable, and standards-compliant code. It includes:

- ✅ Reserved keywords
- 🔤 Standard data types
- ⚙️ Common function blocks
- 💡 Example usage snippets

---

## 🟩 1. 🔠 **Reserved Keywords in IEC 61131-3 Structured Text**

Keywords are reserved words used by the language syntax. They cannot be used as identifiers (e.g., variable names).

### 🔁 Control Flow

| Keyword       | Description |
|---------------|-------------|
| `IF` / `THEN` / `ELSE` / `ELSIF` / `END_IF` | Conditional execution |
| `CASE` / `OF` / `ELSE` / `END_CASE`         | Multi-case selection |
| `FOR` / `TO` / `BY` / `DO` / `END_FOR`      | Loop with counter |
| `WHILE` / `DO` / `END_WHILE`                | Loop while condition is true |
| `REPEAT` / `UNTIL` / `END_REPEAT`           | Loop until condition is true |
| `RETURN`                                     | Exit from a function or method |
| `EXIT`                                       | Exit from current loop |
| `CONTINUE`                                   | Skip to next iteration of loop |

### 🧮 Operators and Logic

| Keyword       | Description |
|---------------|-------------|
| `AND`, `OR`, `NOT`, `XOR` | Logical operators |
| `MOD`                     | Modulo operator |
| `SHL`, `SHR`              | Bitwise shift left/right |
| `ROL`, `ROR`              | Rotate bits left/right |

### 📦 Data Type and Declaration

| Keyword       | Description |
|---------------|-------------|
| `VAR` / `END_VAR`        | Variable declaration block |
| `CONSTANT`               | Constant value declaration |
| `RETAIN`                 | Persistent variable across resets |
| `AT`                     | Assign variable to specific memory address |
| `STRUCT` / `END_STRUCT`  | Define structure type |
| `ARRAY`                  | Define array type |
| `FUNCTION` / `END_FUNCTION` | Define reusable functions |
| `FUNCTION_BLOCK` / `END_FUNCTION_BLOCK` | Define reusable logic with internal state |
| `METHOD` / `END_METHOD`  | Object-oriented method inside a class |
| `CLASS` / `END_CLASS`    | Define object-oriented classes |
| `EXTENDS`                | Inherit from another class |
| `INTERFACE` / `END_INTERFACE` | Define interface contracts |
| `IMPLEMENT`              | Implement an interface |

---

## 🟨 2. 🧾 **Standard Data Types**

IEC 61131-3 defines both elementary and derived data types.

### 🔢 Elementary Data Types

| Type     | Size | Range / Format | Example |
|----------|------|----------------|---------|
| `BOOL`   | 1 bit | TRUE/FALSE     | `bFlag := TRUE;` |
| `BYTE`   | 8 bits | 0..255         | `byData := 16#FF;` |
| `WORD`   | 16 bits | 0..65535       | `wValue := 16#A001;` |
| `DWORD`  | 32 bits | 0..4294967295  | `dwValue := 16#FFFF0000;` |
| `SINT`   | 8 bits | -128..+127     | `siValue := -50;` |
| `INT`    | 16 bits | -32768..+32767 | `iCount := 100;` |
| `DINT`   | 32 bits | -2147483648..+2147483647 | `diTotal := 123456;` |
| `REAL`   | 32 bits | Floating point (~±1e-45 to ±3.4e38) | `rTemp := 25.5;` |
| `LREAL`  | 64 bits | Double precision float | `lrBig := 3.1415926535;` |
| `CHAR`   | 8 bits | ASCII character | `cLetter := 'A';` |
| `STRING(n)` | n chars | Up to n characters | `sName := 'Valve1';` |
| `WCHAR`  | 16 bits | Unicode char (optional) | `wcChar := L'Ω';` |
| `WSTRING(n)` | n WCHARs | Unicode string | `wsText := L'中文';` |
| `TIME`   | - | Duration: T#1ms to T#24d20h31m23s647ms | `tDelay := T#5s;` |
| `DATE`   | - | Date: D#1970-01-01 to D#2106-02-07 | `dtToday := D#2025-06-11;` |
| `TIME_OF_DAY` | - | Time of day: TOD#00:00:00.000 to TOD#23:59:59.999 | `todNow := TOD#14:30:00;` |
| `DATE_AND_TIME` | - | Combined date and time | `dtNow := DT#2025-06-11-14:30:00;` |

---

## 🟦 3. ⚙️ **Common Function Blocks and Functions**

These are standard function blocks available in most IEC 61131-3 environments.

### 🕒 Timer Blocks

| Block | Inputs | Outputs | Description |
|-------|--------|---------|-------------|
| `TON` | `IN`, `PT` | `Q`, `ET` | On-Delay Timer |
| `TOF` | `IN`, `PT` | `Q`, `ET` | Off-Delay Timer |
| `TP`  | `IN`, `PT` | `Q`, `ET` | Pulse Timer |
| `TRIG` | `IN`, `EDGE` | `OUT` | Edge detection |

**Example – TON Usage:**
```pascal
tmrHeatUp(IN := bStartHeating, PT := T#10s);
IF tmrHeatUp.Q THEN
    // Heating complete
END_IF;
```

---

### 🔢 Counter Blocks

| Block | Inputs | Outputs | Description |
|-------|--------|---------|-------------|
| `CTU` | `CU`, `R`, `PV` | `Q`, `CV` | Count Up |
| `CTD` | `CD`, `LD`, `PV` | `Q`, `CV` | Count Down |
| `CTUD` | `CU`, `CD`, `R`, `LD`, `PV` | `QU`, `QD`, `CV` | Up/Down Counter |

**Example – CTU Usage:**
```pascal
ctParts(CU := bPartDetected, R := bResetCounter, PV := 100);
IF ctParts.CV >= 100 THEN
    bBatchComplete := TRUE;
END_IF;
```

---

### 📈 Math Functions

| Function | Inputs | Output | Description |
|----------|--------|--------|-------------|
| `ABS(x)` | x (any numeric) | Same type | Absolute value |
| `SQRT(x)` | x (REAL/LREAL) | Same type | Square root |
| `LN(x)` | x > 0 | REAL | Natural log |
| `EXP(x)` | x | REAL | e^x |
| `SIN(x)` / `COS(x)` / `TAN(x)` | x in radians | REAL | Trigonometric functions |
| `ASIN(x)` / `ACOS(x)` / `ATAN(x)` | x ∈ [-1,1] | REAL | Inverse trigonometric |
| `ATAN2(y,x)` | y, x | REAL | Full-circle arctangent |
| `MAX(x,y)` | x, y | Same type | Returns maximum |
| `MIN(x,y)` | x, y | Same type | Returns minimum |
| `LIMIT(min,val,max)` | min, val, max | Same type | Clamps value between limits |

**Example – LIMIT Usage:**
```pascal
rClamped := LIMIT(0.0, rInput, 100.0);
```

---

### 🧮 Conversion Functions

| Function | Inputs | Output | Description |
|----------|--------|--------|-------------|
| `INT_TO_REAL(i)` | INT | REAL | Converts integer to real |
| `REAL_TO_INT(r)` | REAL | INT | Rounds to nearest integer |
| `STRING_TO_INT(s)` | STRING | INT | Parses string to number |
| `INT_TO_STRING(i)` | INT | STRING | Converts integer to string |
| `DT_TO_DATE(dt)` | DATE_AND_TIME | DATE | Extracts date part |
| `DT_TO_TOD(dt)` | DATE_AND_TIME | TIME_OF_DAY | Extracts time part |

**Example – INT_TO_REAL:**
```pascal
rValue := INT_TO_REAL(iRaw);
```

---

## 🟧 4. 💡 **Code Snippets and Examples**

### 🔁 For Loop
```pascal
FOR i := 1 TO 10 DO
    aValues[i] := i * 2;
END_FOR;
```

### 🔄 While Loop
```pascal
WHILE NOT bDone DO
    IF CheckSensor() THEN
        bDone := TRUE;
    END_IF;
END_WHILE;
```

### 📌 Case Statement
```pascal
CASE iStep OF
    1: StartProcess();
    2: RunHeater();
    3: StopSystem();
    ELSE: HandleError();
END_CASE;
```

### 🛑 If Statement
```pascal
IF fTemperature > 100.0 THEN
    RaiseAlarm();
ELSIF fTemperature < 20.0 THEN
    StartHeater();
ELSE
    MonitorSystem();
END_IF;
```

### 🎛 Method Call
```pascal
oValve.SetPosition(50.0);
```

---

## 🟥 5. 🧭 Best Practices Summary

| Practice | Description |
|---------|-------------|
| **Avoid `LOOP` constructs** | Use scan-cycle-based logic instead |
| **Use meaningful names** | E.g., `bMotorRunning` instead of `flag1` |
| **Group related variables** | Use `STRUCT`s for logical grouping |
| **Limit scope** | Declare variables only where needed |
| **Comment complex logic** | Explain intent, not just what it does |
| **Use helper functions** | Reduce duplication with reusable methods |
| **Test each module** | Isolate and test logic independently |

---

## 🟪 Next Steps

Would you like this reference exported into:
- A downloadable **PDF version**?
- An **IDE snippet pack** (CoDeSys, TwinCAT, etc.)?
- A **wiki page template** for internal documentation?
- A **cheat sheet** format for quick lookup?

Let me know — I can tailor this to your team’s workflow!
