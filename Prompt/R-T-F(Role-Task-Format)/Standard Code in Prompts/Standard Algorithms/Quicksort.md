```
FUNCTION_BLOCK QuickSort
VAR_INPUT
    arr : ARRAY [1..100] OF INT; // The array to be sorted
END_VAR
VAR_OUTPUT
    sortedArr : ARRAY [1..100] OF INT; // The sorted array
END_VAR
VAR
    low : INT := 1; // Lower bound of the segment to be sorted
    high : INT := 100; // Upper bound of the segment to be sorted
    pivot : INT; // Pivot element
    i : INT := 1; // Index of the smaller element
    stack : ARRAY [1..100] OF STRUCT start : INT; end : INT; END_STRUCT; // Stack for recursion simulation
    top : INT := 0; // Top of the stack
    temp : INT; // Temporary variable for swapping
END_VAR

// Initial push to the stack
stack[top].start := low;
stack[top].end := high;
top := top + 1;

// Loop until the stack is empty
WHILE top > 0 DO
    // Pop from the stack
    top := top - 1;
    high := stack[top].end;
    low := stack[top].start;

    // Partition the array
    pivot := Partition(low, high, arr);

    // Push the lower part to the stack if it needs sorting
    IF pivot - 1 > low THEN
        top := top + 1;
        stack[top].start := low;
        stack[top].end := pivot - 1;
    END_IF;

    // Push the higher part to the stack if it needs sorting
    IF pivot + 1 < high THEN
        top := top + 1;
        stack[top].start := pivot + 1;
        stack[top].end := high;
    END_IF;
END_WHILE;

// Copy the sorted array back to the output variable
sortedArr := arr;

RETURN;

END_FUNCTION_BLOCK

// Helper function for partitioning the array
PROCEDURE Partition(low : INT; high : INT; BY_REF arr : ARRAY [1..100] OF INT) : INT
VAR
    pivot : INT := arr[high]; // Choose the last element as pivot
    i : INT := low - 1; // Index of smaller element
    j : INT := low; // Index of current element
    temp : INT; // Temporary variable for swapping
BEGIN
    WHILE j < high DO
        // If current element is smaller than or equal to pivot
        IF arr[j] <= pivot THEN
            i := i + 1;
            // Swap arr[i] and arr[j]
            temp := arr[i];
            arr[i] := arr[j];
            arr[j] := temp;
        END_IF;
        j := j + 1;
    END_WHILE;
    
    // Swap arr[i+1] and arr[high] (or pivot)
    temp := arr[i + 1];
    arr[i + 1] := arr[high];
    arr[high] := temp;
    
    RETURN i + 1; // Return the pivot index
END_PROCEDURE

PROGRAM ExampleProgram
VAR
    unsortedArr : ARRAY [1..100] OF INT; // Unsorted array
    sortedArr : ARRAY [1..100] OF INT; // Sorted array
BEGIN
    // Populate the array with unsorted values
    FOR i := 1 TO 100 DO
        unsortedArr[i] := /* Some value */;
    END_FOR;
    
    // Call the QuickSort function block
    QuickSort(arr:=unsortedArr, sortedArr:=sortedArr);
    
    // Output the sorted array
    FOR i := 1 TO 100 DO
        WRITE(sortedArr[i]);
    END_FOR;
END_PROGRAM
```
