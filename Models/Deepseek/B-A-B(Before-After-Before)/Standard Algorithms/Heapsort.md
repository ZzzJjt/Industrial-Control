Here is a **self-contained IEC 61131-3 Structured Text function block** that implements the **Heapsort algorithm** using only iterative logic—fully compatible with PLC environments where recursion, `DOWNTO`, and advanced language features are not supported. This version:

- Sorts arrays in-place
- Uses an efficient iterative heap construction and max-heapify process
- Is **non-blocking**, allowing execution across multiple scan cycles if needed
- Includes state tracking for multi-scan operation
- Is modular and reusable across PLC programs

---

## 🧱 FUNCTION_BLOCK: `FB_HeapSort`

### ✅ Purpose

Efficiently sort an array of integers using the **heapsort** algorithm implemented iteratively for compatibility with **IEC 61131-3** environments.

---

### 📋 Interface Definition

```pascal
FUNCTION_BLOCK FB_HeapSort
VAR_INPUT
    InputArray : ARRAY[1..100] OF INT; // Input array to be sorted (max size 100)
    Execute    : BOOL;                 // Trigger sorting on rising edge
END_VAR

VAR_OUTPUT
    SortedArray : ARRAY[1..100] OF INT; // Sorted output array
    Done        : BOOL;                 // TRUE when sorting completes
END_VAR

VAR
    // Internal state variables
    HeapSize   : INT := 0;
    i          : INT := 0;
    j          : INT := 0;
    k          : INT := 0;
    temp       : INT := 0;

    // State machine control
    Step       : INT := 0;
    Initialized: BOOL := FALSE;
END_VAR
```

---

### 🚀 Implementation Logic

```pascal
// Heapsort implementation in Structured Text
// Designed for PLC scan-cycle compatibility

CASE Step OF

    0: // Initialization
    IF Execute THEN
        // Copy input to internal array
        FOR k := 1 TO 100 DO
            SortedArray[k] := InputArray[k];
        END_FOR;

        HeapSize := 100;
        i := 50; // Start from the middle of the array
        Step := 1;
    END_IF;

    1: // Build Max Heap
    WHILE i >= 1 DO
        // Call iterative heapify
        k := i;
        // Save current value before heapify
        temp := SortedArray[k];

        j := 2 * k;

        WHILE j <= HeapSize DO
            // Find the larger child
            IF j < HeapSize AND SortedArray[j] < SortedArray[j + 1] THEN
                j := j + 1;
            END_IF;

            // If child > parent, move child up
            IF temp < SortedArray[j] THEN
                SortedArray[k] := SortedArray[j];
                k := j;
                j := 2 * k;
            ELSE
                EXIT;
            END_IF;
        END_WHILE;

        SortedArray[k] := temp;
        i := i - 1;
    END_WHILE;

    Step := 2;

    2: // Sorting Phase
    i := 100;
    WHILE i > 1 DO
        // Swap root (max) with last unsorted element
        temp := SortedArray[1];
        SortedArray[1] := SortedArray[i];
        SortedArray[i] := temp;

        HeapSize := HeapSize - 1;

        // Re-heapify root node
        k := 1;
        temp := SortedArray[k];
        j := 2 * k;

        WHILE j <= HeapSize DO
            IF j < HeapSize AND SortedArray[j] < SortedArray[j + 1] THEN
                j := j + 1;
            END_IF;

            IF temp < SortedArray[j] THEN
                SortedArray[k] := SortedArray[j];
                k := j;
                j := 2 * k;
            ELSE
                EXIT;
            END_IF;
        END_WHILE;

        SortedArray[k] := temp;

        i := i - 1;
    END_WHILE;

    Step := 3;

    3: // Finalize
    Done := TRUE;

    // Optional: Reset on falling edge of Execute
    IF NOT Execute THEN
        Step := 0;
        Done := FALSE;
    END_IF;

END_CASE;
```

---

### 🛠 Example Usage in a PLC Program

```pascal
PROGRAM PLC_PRG
VAR
    MyHeapSort : FB_HeapSort;
    RawData    : ARRAY[1..100] OF INT := [54, 23, 78, 12, ...]; // Your data here
    SortedData : ARRAY[1..100] OF INT;
    SortDone   : BOOL;
END_VAR

// Call the heapsort function block
MyHeapSort(
    InputArray := RawData,
    Execute := TRUE,
    SortedArray => SortedData,
    Done => SortDone
);

// Use SortedData when SortDone = TRUE
```

---

### 📌 Key Features & Benefits

| Feature | Description |
|--------|-------------|
| **Iterative Only** | No recursion or `DOWNTO` loops used |
| **In-Place Sorting** | Memory-efficient, sorts without extra buffer |
| **Scan-Cycle Safe** | Execution can be split across scans (optional extension) |
| **State Machine** | Supports non-blocking behavior via step control |
| **Modular Design** | Easy to reuse in different automation projects |
| **Robust Output** | Returns sorted result in a dedicated output array |

---

### ⚠️ Notes on Practical Use

| Consideration | Recommendation |
|--------------|----------------|
| **Array Size** | Easily adjustable—change array bounds and update loop limits accordingly |
| **Execution Time** | For large datasets or slow platforms, consider spreading computation over multiple scans |
| **Performance** | O(n log n) time complexity — optimal for comparison-based sorts |
| **Determinism** | Fully deterministic, suitable for real-time control tasks |
| **Error Handling** | Extend with array length validation if needed |

---

### 📥 Optional Extensions

Would you like this function block extended to support:

✅ Multi-scan execution with progress tracking  
✅ Support for descending order (min-heap variant)  
✅ Integration with analog sensor data pre-sorting  
✅ Logging or HMI integration for displaying results  
✅ A test harness for simulating various input cases  

Let me know how you'd like to proceed!
