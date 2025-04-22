```
FUNCTION_BLOCK BinarySearch
VAR_INPUT
    arr : ARRAY [1..100] OF INT; // Array of integers
    value : INT; // Value to search for
END_VAR
VAR_OUTPUT
    index : INT; // Index of the found element, or -1 if not found
END_VAR
VAR
    left : INT := 1; // Left boundary of the search
    right : INT := 100; // Right boundary of the search
    mid : INT; // Midpoint of the search
    found : BOOL := FALSE; // Flag to indicate if the value was found
END_VAR

// Input validation: Check if the array is sorted
FOR i := 1 TO 99 DO
    IF arr[i] > arr[i+1] THEN
        index := -2; // Return -2 to indicate the array is not sorted
        RETURN;
    END_IF;
END_FOR;

WHILE NOT found AND (left <= right) DO
    mid := trunc((left + right) / 2); // Calculate midpoint
    IF arr[mid] = value THEN
        found := TRUE;
        index := mid;
    ELSIF arr[mid] < value THEN
        left := mid + 1; // Adjust left boundary
    ELSE
        right := mid - 1; // Adjust right boundary
    END_IF;
END_WHILE;

IF NOT found THEN
    index := -1; // Indicate the value was not found
END_IF;

RETURN;

END_FUNCTION_BLOCK

PROGRAM ExampleProgram
VAR
    numbers : ARRAY [1..100] OF INT;
    val : INT;
    idx : INT;
BEGIN
    // Populate the array with sorted values
    FOR i := 1 TO 100 DO
        numbers[i] := i;
    END_FOR;

    // Set the value to search for
    val := 50;

    // Call the BinarySearch function block
    BinarySearch(arr:=numbers, value:=val, index:=idx);

    // Output the result
    IF idx >= 0 THEN
        // Element found
        WRITE("Value ", val, " found at index ", idx);
    ELSEIF idx = -1 THEN
        // Element not found
        WRITE("Value ", val, " not found.");
    ELSE
        // Array not sorted
        WRITE("Array is not sorted.");
    END_IF;
END_PROGRAM
```
