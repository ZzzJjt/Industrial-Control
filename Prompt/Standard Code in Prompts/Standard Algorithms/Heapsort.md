```
FUNCTION_BLOCK HeapSort
VAR_INPUT
    arr : ARRAY [1..100] OF INT; // The array to be sorted
END_VAR
VAR_OUTPUT
    sortedArr : ARRAY [1..100] OF INT; // The sorted array
END_VAR
VAR
    n : INT := 100; // The number of elements in the array
    i : INT := 1; // Loop index
    j : INT := 1; // Inner loop index
    temp : INT; // Temporary variable for swap
    heapSize : INT := n; // Size of the heap
END_VAR

// Build the max heap
FOR i TO n/2 DO
    MaxHeapify(i, heapSize, arr);
END_FOR;

// Extract elements from the heap one by one
WHILE heapSize > 1 DO
    // Swap the root (maximum value) of the heap with the last element
    temp := arr[heapSize];
    arr[heapSize] := arr[1];
    arr[1] := temp;
    
    // Decrease the size of the heap by one
    heapSize := heapSize - 1;
    
    // Restore the heap property
    MaxHeapify(1, heapSize, arr);
END_WHILE;

// Copy the sorted array back to the output variable
sortedArr := arr;

RETURN;

// Helper function to maintain the max heap property
PROCEDURE MaxHeapify(rootIndex : INT; heapSize : INT; BY_REF arr : ARRAY [1..100] OF INT)
VAR
    largest : INT := rootIndex; // Initialize largest as root
    l : INT := 2 * rootIndex; // Left child index
    r : INT := 2 * rootIndex + 1; // Right child index
BEGIN
    // If left child exists and is greater than root
    IF l <= heapSize AND arr[l] > arr[largest] THEN
        largest := l;
    END_IF;
    
    // If right child exists and is greater than the largest so far
    IF r <= heapSize AND arr[r] > arr[largest] THEN
        largest := r;
    END_IF;
    
    // Change root if needed
    IF largest <> rootIndex THEN
        // Swap
        temp := arr[rootIndex];
        arr[rootIndex] := arr[largest];
        arr[largest] := temp;
        
        // Recursively heapify the affected sub-tree
        MaxHeapify(largest, heapSize, arr);
    END_IF;
END_PROCEDURE

RETURN;

END_FUNCTION_BLOCK

PROGRAM ExampleProgram
VAR
    unsortedArr : ARRAY [1..100] OF INT; // Unsorted array
    sortedArr : ARRAY [1..100] OF INT; // Sorted array
BEGIN
    // Populate the array with unsorted values
    FOR i := 1 TO 100 DO
        unsortedArr[i] := /* Some value */;
    END_FOR;
    
    // Call the HeapSort function block
    HeapSort(arr:=unsortedArr, sortedArr:=sortedArr);
    
    // Output the sorted array
    FOR i := 1 TO 100 DO
        WRITE(sortedArr[i]);
    END_FOR;
END_PROGRAM
```
