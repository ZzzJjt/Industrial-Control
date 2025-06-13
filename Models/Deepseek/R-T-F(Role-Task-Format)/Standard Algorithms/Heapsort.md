FUNCTION_BLOCK FB_Heapsort
VAR_INPUT
    InputArray : ARRAY[1..100] OF INT; // Input array to be sorted
    Execute : BOOL; // Flag to start the sorting process
END_VAR

VAR_OUTPUT
    SortedArray : ARRAY[1..100] OF INT; // Output sorted array (optional if sorting in-place)
    Done : BOOL := FALSE; // Flag indicating sorting is complete
END_VAR

VAR
    N : INT := 100; // Size of the array
    i : INT;
    j : INT;
    Largest : INT;
    Temp : INT;
END_VAR

// Method to perform the heapify operation iteratively
METHOD Heapify(this : REFERENCE TO FB_Heapsort; StartIndex : INT)
BEGIN
    VAR
        LeftChild : INT;
        RightChild : INT;
        Swap : BOOL;
    END_VAR

    Largest := StartIndex;
    LOOP
        Swap := FALSE;
        LeftChild := 2 * Largest;
        RightChild := 2 * Largest + 1;

        // Check if left child exists and is greater than largest
        IF LeftChild <= this.N AND this.SortedArray[LeftChild] > this.SortedArray[Largest] THEN
            Largest := LeftChild;
            Swap := TRUE;
        END_IF;

        // Check if right child exists and is greater than largest
        IF RightChild <= this.N AND this.SortedArray[RightChild] > this.SortedArray[Largest] THEN
            Largest := RightChild;
            Swap := TRUE;
        END_IF;

        // If largest is not root, swap and continue heapifying
        IF Swap THEN
            Temp := this.SortedArray[Largest];
            this.SortedArray[Largest] := this.SortedArray[StartIndex];
            this.SortedArray[StartIndex] := Temp;
            StartIndex := Largest;
        ELSE
            EXIT;
        END_IF;
    END_LOOP;
END_METHOD

// Method to build the max heap
METHOD BuildMaxHeap(this : REFERENCE TO FB_Heapsort)
BEGIN
    FOR i := this.N DIV 2 DOWNTO 1 DO
        this.Heapify(i);
    END_FOR;
END_METHOD

// Main method to execute the heapsort algorithm
METHOD ExecuteSort(this : REFERENCE TO FB_Heapsort)
BEGIN
    // Copy input array to output array for in-place sorting
    FOR i := 1 TO this.N DO
        this.SortedArray[i] := this.InputArray[i];
    END_FOR;

    // Build the max heap
    this.BuildMaxHeap();

    // Extract elements from the heap one by one
    FOR i := this.N DOWNTO 2 DO
        // Move current root to end
        Temp := this.SortedArray[1];
        this.SortedArray[1] := this.SortedArray[i];
        this.SortedArray[i] := Temp;

        // Reduce heap size and heapify the root element
        this.N := this.N - 1;
        this.Heapify(1);
    END_FOR;

    // Reset N to original size for future executions
    this.N := 100;
    this.Done := TRUE;
END_METHOD

// Main execution method
METHOD MAIN(this : REFERENCE TO FB_Heapsort) : BOOL
BEGIN
    IF this.Execute THEN
        this.ExecuteSort();
    END_IF;
    RETURN TRUE;
END_METHOD

PROGRAM MainProgram
VAR
    HeapsortFB : FB_Heapsort;
    InputArray : ARRAY[1..100] OF INT := [9, 7, 5, 11, 12, 2, 14, 3, 10, 6]; // Example input array
    SortedArray : ARRAY[1..100] OF INT;
    Done : BOOL;
END_VAR

// Initialize the input array
FOR i := 1 TO 10 DO
    HeapsortFB.InputArray[i] := InputArray[i];
END_FOR;

// Set the execute flag to start sorting
HeapsortFB.Execute := TRUE;

// Call the function block
HeapsortFB();

// Get the sorted array and completion status
SortedArray := HeapsortFB.SortedArray;
Done := HeapsortFB.Done;

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
