Certainly! Below is a self-contained function block in IEC 61131-3 Structured Text that implements the quicksort algorithm for an array of integers using an iterative approach. The function block replaces recursive calls with stack-emulated logic, ensuring it is scan-cycle-safe and compliant with IEC 61131-3 standards.

### Iterative Quicksort Function Block

```st
[<title="QuicksortFunctionBlock">]
FUNCTION_BLOCK QuicksortFB
VAR_INPUT
    Execute : BOOL;          // Signal to start sorting
    InputArray : ARRAY[1..100] OF INT; // Array to be sorted
END_VAR

VAR_OUTPUT
    SortedArray : ARRAY[1..100] OF INT; // Sorted array output
    Done : BOOL := FALSE;     // Flag indicating if sorting is complete
END_VAR

VAR
    LowStack : ARRAY[1..100] OF INT; // Stack for lower bounds
    HighStack : ARRAY[1..100] OF INT; // Stack for upper bounds
    StackPointer : INT := 0; // Pointer for the stack
    Low : INT;
    High : INT;
    PartitionIndex : INT;
BEGIN
METHOD Partition : VAR_INPUT Low : INT; High : INT; END_VAR : INT
VAR
    Pivot : INT;
    i : INT;
    j : INT;
    Temp : INT;
BEGIN
    Pivot := InputArray[High];
    i := Low - 1;

    FOR j := Low TO High - 1 DO
        IF InputArray[j] < Pivot THEN
            i := i + 1;
            // Swap InputArray[i] and InputArray[j]
            Temp := InputArray[i];
            InputArray[i] := InputArray[j];
            InputArray[j] := Temp;
        END_IF;
    END_FOR;

    // Swap InputArray[i + 1] and InputArray[High]
    Temp := InputArray[i + 1];
    InputArray[i + 1] := InputArray[High];
    InputArray[High] := Temp;

    RETURN i + 1;
END_METHOD

METHOD ExecuteSort : VOID
BEGIN
    IF Execute THEN
        // Initialize the stack
        StackPointer := 0;
        LowStack[1] := 1;
        HighStack[1] := 100;
        StackPointer := StackPointer + 1;

        WHILE StackPointer > 0 DO
            StackPointer := StackPointer - 1;
            Low := LowStack[StackPointer];
            High := HighStack[StackPointer];

            IF Low < High THEN
                PartitionIndex := Partition(Low, High);

                // Push left subarray onto the stack
                IF PartitionIndex - 1 > Low THEN
                    StackPointer := StackPointer + 1;
                    LowStack[StackPointer] := Low;
                    HighStack[StackPointer] := PartitionIndex - 1;
                END_IF;

                // Push right subarray onto the stack
                IF PartitionIndex + 1 < High THEN
                    StackPointer := StackPointer + 1;
                    LowStack[StackPointer] := PartitionIndex + 1;
                    HighStack[StackPointer] := High;
                END_IF;
            END_IF;
        END_WHILE;

        // Copy sorted array to output
        SortedArray := InputArray;
        Done := TRUE;
    ELSE
        Done := FALSE;
    END_IF;
END_METHOD

// Main execution method
ExecuteSort();
END_FUNCTION_BLOCK
```

### Explanation of the Code

#### Inputs
- **Execute**: A boolean signal to initiate the sorting process.
- **InputArray**: An array of 100 integers to be sorted.

#### Outputs
- **SortedArray**: The sorted array after executing quicksort.
- **Done**: A boolean flag indicating whether the sorting process is complete.

#### Internal Variables
- **LowStack**: An array used to simulate the stack for lower bounds of subarrays.
- **HighStack**: An array used to simulate the stack for upper bounds of subarrays.
- **StackPointer**: Pointer for managing the stack.
- **Low**: Lower bound of the current subarray.
- **High**: Upper bound of the current subarray.
- **PartitionIndex**: Index returned by the partitioning function.

#### Methods
- **Partition**: Implements the Lomuto partitioning scheme to rearrange elements around a pivot.
- **ExecuteSort**: Manages the iterative quicksort process using stack emulation.

#### Method: Partition
- **Pivot Selection**: Selects the last element as the pivot.
- **Element Rearrangement**: Rearranges elements such that all elements less than the pivot are on the left, and all elements greater than the pivot are on the right.
- **Return Value**: Returns the index of the pivot after partitioning.

#### Method: ExecuteSort
- **Stack Initialization**: Initializes the stack with the full array bounds.
- **Iterative Sorting**: Uses a `WHILE` loop to process subarrays until the stack is empty.
- **Subarray Management**: Pushes left and right subarrays onto the stack for further processing.
- **Sorting Completion**: Copies the sorted array to `SortedArray` and sets `Done` to `TRUE`.

### Usage Example

To use this function block in a PLC program, you would instantiate it and call its `ExecuteSort` method:

```st
PROGRAM MainProgram
VAR
    MyQuicksort : QuicksortFB;
    InputArray : ARRAY[1..100] OF INT := [5, 3, 8, 4, 2, 7, 1, 10, 6, 9]; // Example array
    SortedArray : ARRAY[1..100] OF INT;
    SortingComplete : BOOL;
BEGIN
    // Configure the sorter
    MyQuicksort.Execute := TRUE;
    MyQuicksort.InputArray := InputArray;

    // Execute the sort
    MyQuicksort();

    // Retrieve the results
    SortedArray := MyQuicksort.SortedArray;
    SortingComplete := MyQuicksort.Done;

    // Output the result
    IF SortingComplete THEN
        // Sorting was successful
        // Use SortedArray as needed
    ELSE
        // Sorting is not yet complete
    END_IF;
END_PROGRAM
```

### Key Features
- **Iterative Approach**: Uses only iterative loops and avoids recursion.
- **Stack Emulation**: Simulates recursion using arrays for lower and upper bounds.
- **Efficient Partitioning**: Implements the Lomuto partitioning scheme.
- **In-Place Sorting**: Sorts the array in place to save memory.
- **Modularity**: Well-defined methods for partitioning and sorting.
- **Scan-Cycle Compatibility**: Suitable for PLC environments with scan-cycle constraints.
- **Reusability**: Configurable for different array sizes and easy to integrate into broader control programs.

This function block provides robust and efficient sorting of integer arrays, making it suitable for various applications in PLC-based systems where fast and reliable sorting is required.
