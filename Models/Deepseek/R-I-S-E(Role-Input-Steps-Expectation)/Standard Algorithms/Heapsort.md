PROGRAM MainProgram
VAR
    HSFB : HeapsortFB;
    InputArray : ARRAY[1..100] OF INT := [99, 87, 76, 65, 54, 43, 32, 21, 10, 1];
    SortedArray : ARRAY[1..100] OF INT;
    Done : BOOL;
    i : INT;
END_VAR

// Configure the function block with a sample input array
HSFB(InputArray := InputArray, Execute := TRUE);

// Call the function block
HSFB.Execute();

// Use the outputs
SortedArray := HSFB.SortedArray;
Done := HSFB.Done;

// Example usage: Print or log the sorted array
IF Done THEN
    FOR i := 1 TO 10 DO
        // Process each element of the sorted array
        // Note: In a real PLC environment, you would typically use outputs to control other parts of the system
    END_FOR;
ELSE
    // Handle incomplete sorting
END_IF;
END_PROGRAM
