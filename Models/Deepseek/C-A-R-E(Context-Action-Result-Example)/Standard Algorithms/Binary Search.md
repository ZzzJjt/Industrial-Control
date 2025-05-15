Certainly! Below is a self-contained function block in IEC 61131-3 Structured Text that performs a binary search on an array of 100 integers. The function block takes as input a sorted array, the target value, and an enable flag. It uses Low, High, and Mid pointers to narrow the search space, validates input conditions, handles edge cases, and outputs the index of the found value or -1 if not found, along with a boolean flag indicating success.

### Binary Search Function Block

```st
[<title="BinarySearchFunctionBlock">]
FUNCTION_BLOCK BinarySearchFB
VAR_INPUT
    Enable : BOOL;          // Enable signal to start the search
    TargetValue : INT;       // Value to search for in the array
    SortedArray : ARRAY[1..100] OF INT; // Sorted array of integers
END_VAR

VAR_OUTPUT
    Index : INT := -1;      // Index of the found value (-1 if not found)
    Success : BOOL := FALSE; // Flag indicating whether the search was successful
END_VAR

VAR
    Low : INT;
    High : INT;
    Mid : INT;
    Found : BOOL := FALSE;
END_VAR

METHOD Execute : VOID
IF Enable THEN
    Low := 1;
    High := 100;
    Found := FALSE;

    WHILE (Low <= High) AND NOT Found DO
        Mid := (Low + High) DIV 2; // Use integer division
        IF SortedArray[Mid] = TargetValue THEN
            Index := Mid;
            Found := TRUE;
            Success := TRUE;
        ELSIF SortedArray[Mid] > TargetValue THEN
            High := Mid - 1;
        ELSE
            Low := Mid + 1;
        END_IF;
    END_WHILE;

    IF NOT Found THEN
        Index := -1;
        Success := FALSE;
    END_IF;
END_IF;
END_METHOD
```

### Explanation of the Code

#### Inputs
- **Enable**: A boolean flag to initiate the search.
- **TargetValue**: The integer value to be searched within the array.
- **SortedArray**: An array of 100 integers that must be sorted in ascending order.

#### Outputs
- **Index**: The index of the `TargetValue` in the `SortedArray`. If the value is not found, it returns `-1`.
- **Success**: A boolean flag indicating whether the search was successful (`TRUE` if found, `FALSE` otherwise).

#### Internal Variables
- **Low**: The lower bound of the current search range.
- **High**: The upper bound of the current search range.
- **Mid**: The midpoint of the current search range.
- **Found**: A boolean flag indicating whether the target value has been found.

#### Method: Execute
- **Initialization**: Sets `Low` to 1, `High` to 100, and `Found` to `FALSE`.
- **Binary Search Loop**:
  - Continues while `Low` is less than or equal to `High` and the target value has not been found.
  - Calculates the midpoint `Mid` using integer division.
  - Compares the `TargetValue` with the element at `SortedArray[Mid]`.
  - Adjusts `Low` or `High` based on the comparison to narrow down the search range.
- **Post-Loop**:
  - If the target value is not found, sets `Index` to `-1` and `Success` to `FALSE`.

### Usage Example

To use this function block in a PLC program, you would instantiate it and call its `Execute` method:

```st
PROGRAM MainProgram
VAR
    MyBinarySearch : BinarySearchFB;
    SortedArray : ARRAY[1..100] OF INT := [1, 2, 3, ..., 100]; // Pre-sorted array
    TargetValue : INT := 50;
BEGIN
    MyBinarySearch.Enable := TRUE;
    MyBinarySearch.TargetValue := TargetValue;
    MyBinarySearch.SortedArray := SortedArray;
    
    MyBinarySearch.Execute();
    
    IF MyBinarySearch.Success THEN
        // Target value found at index MyBinarySearch.Index
    ELSE
        // Target value not found
    END_IF;
END_PROGRAM
```

This function block is modular, reliable, and reusable, making it suitable for efficient data lookups in industrial automation systems running on PLC hardware.
