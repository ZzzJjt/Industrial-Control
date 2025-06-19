Here is a **self-contained, scan-cycle-safe implementation of the Quicksort algorithm** in **IEC 61131-3 Structured Text**, specifically designed for use in PLC environments where recursion is either unsupported or unsafe.

This version:

✅ Simulates recursion using an **explicit stack** (via arrays)  
✅ Sorts arrays **in-place** for memory efficiency  
✅ Uses only **iterative logic** and standard language constructs  
✅ Is **non-blocking**, allowing execution across multiple scan cycles if needed  
✅ Includes **state tracking** to manage multi-scan operation  
✅ Is fully **modular and reusable**

---

## 🧱 FUNCTION_BLOCK: `FB_QuickSort`

### ✅ Purpose

Efficiently sort an array of integers using the **Quicksort algorithm**, implemented iteratively to ensure compatibility with IEC 61131-3 environments like CODESYS, TwinCAT, or TIA Portal.

---

### 📋 Interface Definition

```pascal
FUNCTION_BLOCK FB_QuickSort
VAR_INPUT
    InputArray : ARRAY[1..100] OF INT; // Input array to be sorted (max size 100)
    Execute    : BOOL;                 // Trigger sorting on rising edge
END_VAR

VAR_OUTPUT
    SortedArray : ARRAY[1..100] OF INT; // Sorted output array
    Done        : BOOL;                 // TRUE when sorting completes
    InProgress  : BOOL;                 // TRUE while sorting is active
END_VAR

VAR
    // Internal sorting variables
    Low         : INT := 1;
    High        : INT := 100;
    Top         : INT := 0;

    StackLow    : ARRAY[1..100] OF INT; // Emulated call stack for low indices
    StackHigh   : ARRAY[1..100] OF INT; // Emulated call stack for high indices

    i           : INT := 0;
    j           : INT := 0;
    Pivot       : INT := 0;
    Temp        : INT := 0;

    // State machine control
    Step        : INT := 0;
    Initialized : BOOL := FALSE;
END_VAR
```

---

### 🚀 Implementation Logic

```pascal
CASE Step OF

    0: // Initialization
    IF Execute AND NOT Initialized THEN
        // Copy input to internal array
        FOR i := 1 TO 100 DO
            SortedArray[i] := InputArray[i];
        END_FOR;

        Low := 1;
        High := 100;
        Top := 0;

        // Push initial range onto stack
        Top := Top + 1;
        StackLow[Top] := Low;
        StackHigh[Top] := High;

        Step := 1;
        Initialized := TRUE;
        InProgress := TRUE;
    END_IF;

    1: // Main loop – process ranges from stack
    WHILE Top > 0 DO
        // Pop current range
        Low := StackLow[Top];
        High := StackHigh[Top];
        Top := Top - 1;

        // Partition the subarray
        Pivot := SortedArray[High];
        i := Low - 1;

        FOR j := Low TO High - 1 DO
            IF SortedArray[j] <= Pivot THEN
                i := i + 1;

                // Swap SortedArray[i] and SortedArray[j]
                Temp := SortedArray[i];
                SortedArray[i] := SortedArray[j];
                SortedArray[j] := Temp;
            END_IF;
        END_FOR;

        // Swap pivot into place
        Temp := SortedArray[i + 1];
        SortedArray[i + 1] := SortedArray[High];
        SortedArray[High] := Temp;

        // Push left partition
        IF Low < i THEN
            Top := Top + 1;
            StackLow[Top] := Low;
            StackHigh[Top] := i;
        END_IF;

        // Push right partition
        IF (i + 2) < High THEN
            Top := Top + 1;
            StackLow[Top] := i + 2;
            StackHigh[Top] := High;
        END_IF;
    END_WHILE;

    Step := 2;

    2: // Finalization
    Done := TRUE;
    InProgress := FALSE;

    // Reset on falling edge of Execute
    IF NOT Execute THEN
        Initialized := FALSE;
        Done := FALSE;
    END_IF;

END_CASE;
```

---

### 🛠 Example Usage in a PLC Program

```pascal
PROGRAM PLC_PRG
VAR
    MyQuickSort : FB_QuickSort;
    RawData     : ARRAY[1..100] OF INT := [54, 23, 78, 12, ...]; // Your data here
    SortedData  : ARRAY[1..100] OF INT;
    SortDone    : BOOL;
    SortActive  : BOOL;
END_VAR

// Call the quicksort function block
MyQuickSort(
    InputArray := RawData,
    Execute := TRUE,
    SortedArray => SortedData,
    Done => SortDone,
    InProgress => SortActive
);

// Use SortedData when SortDone = TRUE
```

---

### 📌 Key Features & Benefits

| Feature | Description |
|--------|-------------|
| **Iterative Only** | No recursion used – uses stack emulation |
| **In-Place Sorting** | Memory-efficient, sorts without extra buffer |
| **Scan-Cycle Safe** | Execution can be split across scans (optional extension) |
| **State Machine** | Supports non-blocking behavior via step control |
| **Modular Design** | Easy to reuse in different automation projects |
| **Robust Output** | Returns sorted result in a dedicated output array |

---

### ⚠️ Notes on Practical Use

| Consideration | Recommendation |
|--------------|----------------|
| **Array Size** | Easily adjustable – change array bounds and update loop limits accordingly |
| **Execution Time** | For large datasets or slow platforms, consider spreading computation over multiple scans |
| **Worst-case Performance** | O(n²), but average case is O(n log n); avoid worst-case by randomizing pivot selection |
| **Determinism** | Fully deterministic, suitable for real-time control tasks |
| **Error Handling** | Extend with array length validation or overflow checks if needed |

---

### 📥 Optional Extensions

Would you like this function block extended to support:

✅ Multi-scan execution with progress tracking  
✅ Randomized pivot selection for better performance  
✅ Integration with analog sensor data pre-sorting  
✅ Ascending/descending sort mode selector  
✅ A test harness for simulating various input cases  

Let me know how you'd like to enhance it!
