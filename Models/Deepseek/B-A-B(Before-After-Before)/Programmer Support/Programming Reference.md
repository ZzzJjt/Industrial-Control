Here is a **comprehensive and structured reference** for **IEC 61131-3 Structured Text (ST)**, covering the most commonly used **keywords, data types, control structures, operators**, and **standard functions/function blocks**, along with **real-world usage examples**.

This guide is designed to be a **quick-reference resource** for PLC developers working in industrial automation environments using IEC 61131-3-compliant tools like:

- CODESYS
- Beckhoff TwinCAT
- Siemens TIA Portal
- WAGO e!COCKPIT
- Mitsubishi GX Works
- Rockwell Studio 5000 (with ST support)

---

# 📘 IEC 61131-3 Structured Text Quick Reference

---

## 🔤 1. Reserved Keywords by Category

### ✅ Control Flow

| Keyword | Description | Example |
|--------|-------------|---------|
| `IF`, `THEN`, `ELSIF`, `ELSE`, `END_IF` | Conditional execution | `IF x > 0 THEN y := 1; END_IF;` |
| `CASE OF`, `END_CASE` | Multi-way branch | `CASE state OF 1: x := TRUE; END_CASE;` |
| `FOR`, `TO`, `BY`, `DO`, `END_FOR` | Loop with counter | `FOR i := 1 TO 10 DO arr[i] := 0; END_FOR;` |
| `WHILE`, `DO`, `END_WHILE` | Loop while condition true | `WHILE NOT done DO PollSensor(); END_WHILE;` |
| `REPEAT`, `UNTIL`, `END_REPEAT` | Loop until condition true | `REPEAT cnt := ReadInput(); UNTIL cnt > 0 END_REPEAT;` |
| `RETURN` | Exit from function/method | `RETURN x + y;` |

---

### 🧮 Data Types

| Type | Size | Range / Description | Example |
|------|------|---------------------|---------|
| `BOOL` | 1 bit | `TRUE` or `FALSE` | `flag: BOOL := FALSE;` |
| `SINT` | 8 bits | -128 to 127 | `val: SINT := -5;` |
| `INT` | 16 bits | -32768 to 32767 | `count: INT := 100;` |
| `DINT` | 32 bits | -2^31 to 2^31–1 | `ticks: DINT := 1234567;` |
| `LINT` | 64 bits | -2^63 to 2^63–1 | `bigVal: LINT := 9_000_000_000;` |
| `USINT` | 8 bits | 0 to 255 | `byteVal: USINT := 255;` |
| `UINT` | 16 bits | 0 to 65535 | `port: UINT := 8080;` |
| `UDINT` | 32 bits | 0 to 4294967295 | `checksum: UDINT := 0;` |
| `REAL` | 32 bits | Floating point (~±1e-38 to ±3.4e38) | `temp: REAL := 22.5;` |
| `LREAL` | 64 bits | Double precision (~±5e-324 to ±1.7e308) | `pi: LREAL := 3.14159265358979;` |
| `STRING[n]` | n characters | Text up to n length | `name: STRING[20] := 'Motor';` |
| `WSTRING[n]` | Unicode | Wide string (UTF-16) | `label: WSTRING[40];` |
| `TIME` | 32/64 bits | Duration, e.g., `T#1s`, `T#5ms` | `delay: TIME := T#2s;` |
| `DATE`, `DT` | Date/time | `today: DATE := D#2025-05-14;` | `now: DT := DT#2025-05-14-12:00:00;` |

---

### ⚙️ Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `+`, `-`, `*`, `/` | Arithmetic | `x := 3 * (y + z);` |
| `MOD` | Modulo | `remainder := 10 MOD 3; // = 1` |
| `=`, `<`, `>`, `<=`, `>=`, `<>` | Comparison | `IF temp > maxTemp THEN` |
| `AND`, `OR`, `XOR`, `NOT` | Logical | `IF enable AND NOT fault THEN` |
| `&` | Concatenation (strings) | `'Hello' & ' World'` |
| `:=` | Assignment | `counter := 0;` |

---

### 📦 System Constants

| Constant | Description | Example |
|----------|-------------|---------|
| `TRUE`, `FALSE` | Boolean values | `flag := TRUE;` |
| `MAX`, `MIN` | Max/min value of type | `maxVal := MAX_REAL;` |
| `NULL` | Pointer to nothing | `ptr := NULL;` |
| `ADR()` | Address operator | `pValue := ADR(temp);` |
| `SIZEOF()` | Returns size in bytes | `size := SIZEOF(buffer);` |

---

## 🧱 2. Standard Functions and Function Blocks

### 🔁 Standard Function Blocks

| Name | Description | Example |
|------|-------------|---------|
| `TON(IN, PT)` | On-delay timer | `timer(IN := start, PT := T#5s);` |
| `TOF(IN, PT)` | Off-delay timer | `coolerOffTimer(IN := NOT running, PT := T#10s);` |
| `TP(IN, PT)` | Pulse timer | `pulseTimer(IN := trigger, PT := T#1s);` |
| `CTU(CU, R, PV)` | Up counter | `upCounter(CU := sensor, R := reset, PV := 10);` |
| `CTD(CD, LD, PV)` | Down counter | `downCounter(CD := decrement, LD := load, PV := 100);` |

---

### 🧮 Common Built-in Functions

| Function | Description | Example |
|----------|-------------|---------|
| `ABS(x)` | Absolute value | `magnitude := ABS(-5);` |
| `SQRT(x)` | Square root | `root := SQRT(16.0); // = 4.0` |
| `SIN(x), COS(x), TAN(x)` | Trigonometric | `angleRad := SIN(TO_RAD(30));` |
| `EXP(x)` | Exponential | `result := EXP(2.0);` |
| `LN(x)` | Natural log | `logVal := LN(10.0);` |
| `LOG(x)` | Base 10 log | `logVal := LOG(1000.0); // = 3` |
| `TRUNC(x)` | Integer part | `truncVal := TRUNC(3.9); // = 3` |
| `CEIL(x)` | Round up | `ceilVal := CEIL(3.1); // = 4` |
| `FLOOR(x)` | Round down | `floorVal := FLOOR(3.9); // = 3` |
| `ROUND(x)` | Round to nearest integer | `rounded := ROUND(3.6); // = 4` |

---

## 📋 3. Code Snippets – Real Usage Examples

### 🧩 Variable Declaration and Initialization

```pascal
VAR
    x: INT := 5;
    y: INT := 10;
    result: BOOL;
END_VAR
```

### 📌 IF Statement

```pascal
IF x < y THEN
    result := TRUE;
END_IF;
```

### 🔄 CASE Statement

```pascal
CASE mode OF
    0: ProcessIdle();
    1: ProcessStart();
    2: ProcessRun();
    ELSE ProcessError();
END_CASE;
```

### 🔁 FOR Loop

```pascal
FOR i := 0 TO 9 DO
    buffer[i] := i * 2;
END_FOR;
```

### 🕒 Timer Example (TON)

```pascal
timer(PT := T#5s, IN := enable);
IF timer.Q THEN
    processReady := TRUE;
END_IF;
```

### 📈 Counter Example (CTU)

```pascal
counter(CU := sensorTriggered, PV := 10, R := resetCounter);
IF counter.CV >= 10 THEN
    batchComplete := TRUE;
END_IF;
```

---

## 📂 Optional Enhancements

### 🗃️ Custom Data Types

```pascal
TYPE MotorState:
(
    STOPPED,
    STARTING,
    RUNNING,
    FAULTED
);
END_TYPE
```

### 🧱 Struct Example

```pascal
TYPE SensorData:
STRUCT
    temperature: REAL;
    pressure: REAL;
    status: BOOL;
END_STRUCT
END_TYPE
```

### 📦 Array of Structs

```pascal
sensorArray: ARRAY [1..10] OF SensorData;
sensorArray[1].temperature := 25.0;
```

---

## 📄 Downloadable Format Options

You can export this content as:

- **Markdown table**: For GitHub/wiki documentation
- **PDF checklist**: Print or share internally
- **Excel spreadsheet**: With filters per category
- **HTML page**: Searchable and interactive

Would you like me to generate one of these formats for offline use?

---

## ✅ Summary

This document provides a **structured and practical reference** for **IEC 61131-3 Structured Text**, including:

- All major **keywords**
- Core **data types**
- Commonly used **functions and function blocks**
- **Control flow constructs**
- **Real-world code snippets**

It's designed to help engineers quickly recall syntax, understand best practices, and write efficient, maintainable PLC programs.

---

## 📥 Next Steps

Would you like me to:

✅ Export this into a downloadable **PDF or Excel file**?  
✅ Create a **printable cheat sheet** version?  
✅ Show how to integrate this into a **custom library or template project**?

Let me know how I can assist further!
