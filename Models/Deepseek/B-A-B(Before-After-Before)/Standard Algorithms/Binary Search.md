Here is a **self-contained IEC 61131-3 Structured Text** implementation of a **binary search function block** for use in PLC-based industrial control systems. This implementation is optimized for performance, includes input validation, and ensures safe array indexing to avoid runtime errors.

---

# 🔍 Binary Search in a 100-Element Array  
## ✅ IEC 61131-3 Structured Text | Reusable Function Block

---

## 📌 Why Use Binary Search?

Binary search offers **logarithmic time complexity (O(log n))**, which is vastly superior to linear search (O(n)) when working with large or sorted datasets—ideal for applications such as:

- Threshold detection
- Data classification
- Sorting index lookup
- Industrial setpoint matching

This makes it especially valuable in real-time PLC systems where fast data access is critical.

---

## 🧱 FUNCTION BLOCK: `FB_BinarySearch`

### Interface

```pascal
FUNCTION_BLOCK FB_BinarySearch
VAR_INPUT
    Enable      : BOOL;                 // Activates the search
    SearchArray : ARRAY[1..100] OF INT; // Sorted input array (ascending order)
    TargetValue : INT;                  // Value to find in the array
END_VAR

VAR_OUTPUT
    Found       : BOOL;                 // TRUE if value found
    Index       : INT := -1;            // Position in array (1-based), -1 if not found
END_VAR

VAR
    Low         : INT := 1;             // Lower bound of search range
    High        : INT := 100;           // Upper bound of search range
    Mid         : INT := 0;             // Midpoint calculation
    SortedOK    : BOOL := TRUE;         // Flag to validate array sort order
END_VAR
```

---

### Implementation Logic

```pascal
// Binary Search Function Block
// Assumes SearchArray is sorted in ascending order
// Validates sorting before search begins

METHOD PRIVATE ValidateSorted:
VAR
    i: INT;
END_VAR
BEGIN
    FOR i := 1 TO 99 DO
        IF SearchArray[i] > SearchArray[i+1] THEN
            SortedOK := FALSE;
            EXIT;
        END_IF;
    END_FOR;
END_METHOD

// Main logic executed only when Enable = TRUE
IF Enable THEN
    // Reset output values
    Found := FALSE;
    Index := -1;

    // Validate that the array is sorted
    ValidateSorted();
    IF NOT SortedOK THEN
        ; // Optionally trigger an error flag or alarm
        RETURN;
    END_IF;

    // Initialize binary search bounds
    Low := 1;
    High := 100;

    // Perform binary search
    WHILE Low <= High AND NOT Found DO
        Mid := (Low + High) \ 2;  // Integer division

        IF SearchArray[Mid] = TargetValue THEN
            Found := TRUE;
            Index := Mid;

        ELSIF SearchArray[Mid] < TargetValue THEN
            Low := Mid + 1;

        ELSE
            High := Mid - 1;

        END_IF;
    END_WHILE;

ELSE
    // If Enable = FALSE, reset internal state
    Low := 1;
    High := 100;
    SortedOK := TRUE;
END_IF;
```

---

## 📋 Example Usage in a PLC Program

```pascal
PROGRAM PLC_PRG
VAR
    MyBinarySearch : FB_BinarySearch;
    TestArray      : ARRAY[1..100] OF INT := [10, 20, 30, 40, 50, ...]; // Must be sorted
    Target         : INT := 40;
    ResultIndex    : INT;
    WasFound       : BOOL;
END_VAR

// Call the function block
MyBinarySearch(
    Enable := TRUE,
    SearchArray := TestArray,
    TargetValue := Target
);

// Output results
WasFound := MyBinarySearch.Found;
ResultIndex := MyBinarySearch.Index;
```

---

## 📌 Key Features & Benefits

| Feature | Description |
|--------|-------------|
| **Efficiency** | O(log n) search time – ideal for time-critical operations |
| **Safety** | Prevents out-of-bounds indexing using strict bounds checking |
| **Validation** | Ensures the array is sorted before proceeding |
| **Modular Design** | Encapsulated logic allows reuse across programs |
| **Clear Feedback** | Outputs indicate success and position |
| **IEC 61131-3 Compliant** | Works in CODESYS, TwinCAT, Beckhoff, Siemens, and other PLC platforms |

---

## ⚠️ Notes on Practical Use

| Consideration | Recommendation |
|--------------|----------------|
| **Sorting Requirement** | Always ensure the array is pre-sorted in ascending order before passing to the function block |
| **Real-Time Safety** | Binary search is deterministic and suitable for cyclic tasks |
| **Error Handling** | Extend with diagnostics or alarms if unsorted array detected |
| **Performance** | Avoid calling every scan cycle unless necessary |
| **Memory Optimization** | For larger arrays, consider pointer-based or indirect addressing techniques |

---

## 📥 Deliverables

Would you like me to provide:

✅ A downloadable `.st` file compatible with CODESYS or other IEC 61131-3 IDEs?  
✅ An extended version supporting **descending order arrays** or **floating-point values**?  
✅ Integration with a **linear search comparison benchmark**?  
✅ A **test harness** for simulating various edge cases (e.g., duplicate values, not found)?

Let me know how you'd like to proceed!
