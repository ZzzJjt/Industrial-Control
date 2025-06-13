FUNCTION_BLOCK FB_Quicksort
VAR_INPUT
    InputArray : ARRAY[1..100] OF INT; // Input array to be sorted
    Execute : BOOL; // Flag to start the sorting process
END_VAR

VAR_OUTPUT
    SortedArray : ARRAY[1..100] OF INT; // Output sorted array (optional if sorting in-place)
    Done : BOOL := FALSE; // Flag indicating sorting is complete
    Busy : BOOL := FALSE; // Flag indicating sorting is in progress
END_VAR

VAR
    LowStack : ARRAY[1..50] OF INT; // Stack for low indices
    HighStack : ARRAY[1..50] OF INT; // Stack for high indices
    StackTop : INT := 0; // Top of the stack
    N : INT := 100; // Size of the array
    Pivot : INT;
    i : INT;
    j : INT;
    Temp : INT;
    Low : INT;
    High : INT;
END_VAR

// Method to perform partitioning using Lomuto's partition scheme
METHOD Partition(this : REFERENCE TO FB_Quicksort; Low : INT; High : INT) : INT
BEGIN
    VAR
        PivotValue : INT;
    END_VAR

    PivotValue := this.SortedArray[High];
    i := Low - 1;

    FOR j := Low TO High - 1 DO
        IF this.SortedArray[j] <= PivotValue THEN
            i := i + 1;
            // Swap elements at i and j
            Temp := this.SortedArray[i];
            this.SortedArray[i] := this.SortedArray[j];
            this.SortedArray[j] := Temp;
        END_IF;
    END_FOR;

    // Swap pivot element with element at i+1
    Temp := this.SortedArray[i + 1];
    this.SortedArray[i + 1] := this.SortedArray[High];
    this.SortedArray[High] := Temp;

    RETURN i + 1;
END_METHOD

// Main method to execute the quicksort algorithm iteratively
METHOD ExecuteSort(this : REFERENCE TO FB_Quicksort)
BEGIN
    VAR
        Low : INT;
        High : INT;
        PivotIndex : INT;
    END_VAR

    // Initialize the stack
    this.StackTop := 0;

    // Push initial subarray bounds onto the stack
    this.StackTop := this.StackTop + 1;
    this.LowStack[this.StackTop] := 1;
    this.HighStack[this.StackTop] := this.N;

    // Loop until the stack is empty
    WHILE this.StackTop > 0 DO
        // Pop subarray bounds from the stack
        High := this.HighStack[this.StackTop];
        Low := this.LowStack[this.StackTop];
        this.StackTop := this.StackTop - 1;

        // Partition the subarray
        PivotIndex := this.Partition(Low, High);

        // If there are elements on the left side of the pivot, push them onto the stack
        IF PivotIndex - 1 > Low THEN
            this.StackTop := this.StackTop + 1;
            this.LowStack[this.StackTop] := Low;
            this.HighStack[this.StackTop] := PivotIndex - 1;
        END_IF;

        // If there are elements on the right side of the pivot, push them onto the stack
        IF PivotIndex + 1 < High THEN
            this.StackTop := this.StackTop + 1;
            this.LowStack[this.StackTop] := PivotIndex + 1;
            this.HighStack[this.StackTop] := High;
        END_IF;
    END_WHILE;

    this.Done := TRUE;
END_METHOD

// Main execution method
METHOD MAIN(this : REFERENCE TO FB_Quicksort) : BOOL
BEGIN
    IF this.Execute AND NOT this.Busy THEN
        this.Busy := TRUE;
        this.Done := FALSE;

        // Copy input array to output array for in-place sorting
        FOR i := 1 TO this.N DO
            this.SortedArray[i] := this.InputArray[i];
        END_FOR;

        // Execute the quicksort algorithm
        this.ExecuteSort();

        this.Busy := FALSE;
    END_IF;
    RETURN TRUE;
END_METHOD

PROGRAM MainProgram
VAR
    QuicksortFB : FB_Quicksort;
    InputArray : ARRAY[1..100] OF INT := [9, 7, 5, 11, 12, 2, 14, 3, 10, 6]; // Example input array
    SortedArray : ARRAY[1..100] OF INT;
    Done : BOOL;
    Busy : BOOL;
END_VAR

// Initialize the input array
FOR i := 1 TO 10 DO
    QuicksortFB.InputArray[i] := InputArray[i];
END_FOR;

// Set the execute flag to start sorting
QuicksortFB.Execute := TRUE;

// Call the function block
QuicksortFB();

// Get the sorted array and status flags
SortedArray := QuicksortFB.SortedArray;
Done := QuicksortFB.Done;
Busy := QuicksortFB.Busy;

// Output the results
IF Done THEN
    // Display or log the sorted array
    // FOR i := 1 TO 10 DO
    //     Display(SortedArray[i]); // Hypothetical display function
    // END_FOR
ELSE
    // Handle incomplete sorting
    // Log("Sorting not yet complete"); // Hypothetical logging function
END_IF;

// Check if sorting is busy
IF Busy THEN
    // Log("Sorting in progress"); // Hypothetical logging function
END_IF;
