FUNCTION_BLOCK FB_BinarySearch
VAR_INPUT
    Enable : BOOL; // Enable flag to start the search
    TargetValue : INT; // Value to search for in the array
    Array : ARRAY[1..100] OF INT; // Sorted array of integers
END_VAR

VAR_OUTPUT
    Index : INT := -1; // Index of the found value, -1 if not found
    Found : BOOL := FALSE; // Boolean indicating if the value was found
END_VAR

VAR
    Low : INT := 1; // Lower boundary of the search range
    High : INT := 100; // Upper boundary of the search range
    Mid : INT; // Middle point of the current search range
    ValidArray : BOOL := TRUE; // Flag to indicate if the array is valid
END_VAR

// Method to perform binary search
METHOD Execute(this : REFERENCE TO FB_BinarySearch) : BOOL
BEGIN
    // Reset output variables
    this.Index := -1;
    this.Found := FALSE;

    // Validate the enable flag
    IF NOT this.Enable THEN
        RETURN FALSE;
    END_IF;

    // Validate the array bounds (ensure all elements are within reasonable range)
    FOR i := 1 TO 100 DO
        IF this.Array[i] < -1000 OR this.Array[i] > 1000 THEN
            this.ValidArray := FALSE;
            RETURN FALSE;
        END_IF;
    END_FOR;

    // Binary search algorithm
    WHILE this.Low <= this.High DO
        this.Mid := (this.Low + this.High) DIV 2;

        IF this.Array[this.Mid] = this.TargetValue THEN
            this.Index := this.Mid;
            this.Found := TRUE;
            EXIT;
        ELSIF this.Array[this.Mid] < this.TargetValue THEN
            this.Low := this.Mid + 1;
        ELSE
            this.High := this.Mid - 1;
        END_IF;
    END_WHILE;

    RETURN TRUE;
END_METHOD

// Main execution method
METHOD MAIN(this : REFERENCE TO FB_BinarySearch) : BOOL
BEGIN
    RETURN this.Execute();
END_METHOD


PROGRAM MainProgram
VAR
    BinarySearchFB : FB_BinarySearch;
    ResultIndex : INT;
    ResultFound : BOOL;
    SortedArray : ARRAY[1..100] OF INT := [ /* Populate with sorted integers */ ];
END_VAR

// Initialize the array with sorted integers
FOR i := 1 TO 100 DO
    SortedArray[i] := i * 2; // Example: 2, 4, 6, ..., 200
END_FOR;

// Call the binary search function block
BinarySearchFB(Enable := TRUE, TargetValue := 50, Array := SortedArray);

// Check results
ResultIndex := BinarySearchFB.Index;
ResultFound := BinarySearchFB.Found;

// Output results
IF ResultFound THEN
    // Do something with the result index
ELSE
    // Handle case where the value was not found
END_IF;
