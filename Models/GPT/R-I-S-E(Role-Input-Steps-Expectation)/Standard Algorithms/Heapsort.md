FUNCTION_BLOCK FB_HeapSort
VAR_INPUT
    Execute : BOOL; // Trigger to start sorting
    InputArray : ARRAY[1..10] OF INT; // Example size N = 10
END_VAR

VAR_OUTPUT
    SortedArray : ARRAY[1..10] OF INT;
    Done : BOOL; // TRUE when sorting is complete
END_VAR

VAR
    HeapSize : INT := 10;
    Phase : INT := 0; // 0: Idle, 1: Build Heap, 2: Sort
    I : INT := 1; // loop counter
    Temp : INT;
    Parent : INT;
    Largest : INT;
    Left : INT;
    Right : INT;
    HeapifyIndex : INT;
    HeapifyNeeded : BOOL;
END_VAR

IF Execute AND NOT Done THEN
    CASE Phase OF

        // Phase 1: Copy input and build max-heap
        0:
            FOR I := 1 TO 10 DO
                SortedArray[I] := InputArray[I];
            END_FOR;
            I := 5; // Start heap building from floor(N/2)
            Phase := 1;

        // Phase 2: Build Heap (Bottom-Up)
        1:
            IF I >= 1 AND I <= 5 THEN
                HeapifyIndex := I;
                HeapifyNeeded := TRUE;
                Phase := 11; // Jump to iterative heapify
            ELSE
                HeapSize := 10;
                I := 10;
                Phase := 2;
            END_IF;

        // Phase 3: Heapsort process
        2:
            IF I > 1 THEN
                // Swap root (max) with end
                Temp := SortedArray[1];
                SortedArray[1] := SortedArray[I];
                SortedArray[I] := Temp;
                HeapSize := I - 1;
                HeapifyIndex := 1;
                HeapifyNeeded := TRUE;
                I := I - 1;
                Phase := 11; // Re-heapify root
            ELSE
                Done := TRUE;
                Phase := 0; // Reset
            END_IF;

        // Phase 11: Iterative Heapify
        11:
            WHILE HeapifyNeeded DO
                Parent := HeapifyIndex;
                Left := 2 * Parent;
                Right := 2 * Parent + 1;
                Largest := Parent;

                IF Left <= HeapSize AND SortedArray[Left] > SortedArray[Largest] THEN
                    Largest := Left;
                END_IF;
                IF Right <= HeapSize AND SortedArray[Right] > SortedArray[Largest] THEN
                    Largest := Right;
                END_IF;

                IF Largest <> Parent THEN
                    // Swap
                    Temp := SortedArray[Parent];
                    SortedArray[Parent] := SortedArray[Largest];
                    SortedArray[Largest] := Temp;
                    HeapifyIndex := Largest; // Continue heapifying
                ELSE
                    HeapifyNeeded := FALSE;
                    IF Phase = 11 AND I >= 1 AND Phase = 1 THEN
                        I := I - 1;
                        Phase := 1; // Go back to build heap
                    ELSE
                        Phase := 2; // Continue sorting
                    END_IF;
                END_IF;
            END_WHILE;
    END_CASE;
END_IF;
