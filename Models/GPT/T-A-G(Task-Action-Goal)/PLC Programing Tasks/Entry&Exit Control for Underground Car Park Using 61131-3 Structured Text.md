VAR
    // Inputs
    X1 : BOOL; // Car sensor at ground floor
    X2 : BOOL; // Car sensor at basement

    // Memory flags (one-scan triggers)
    M1, M2, M3, M4 : BOOL; 
    M20 : BOOL := FALSE; // Passage occupied from ground floor
    M30 : BOOL := FALSE; // Passage occupied from basement

    // Outputs
    Y1 : BOOL := FALSE; // Red light (stop)
    Y2 : BOOL := TRUE;  // Green light (go)
END_VAR

// Memory flag logic (one-scan memory should be externally latched)
// These flags should be set/reset based on edge detection logic externally

// Set passage occupied from ground floor
IF M1 OR M4 THEN
    M20 := TRUE;
END_IF;

// Set passage occupied from basement
IF M2 OR M3 THEN
    M30 := TRUE;
END_IF;

// Clear ground floor occupancy (car exited)
IF M3 OR M4 THEN
    M20 := FALSE;
END_IF;

// Clear basement occupancy (car exited)
IF M1 OR M2 THEN
    M30 := FALSE;
END_IF;

// Light control logic
IF M20 OR M30 THEN
    Y1 := TRUE;  // Red light on (stop)
    Y2 := FALSE; // Green light off
ELSE
    Y1 := FALSE; // Red light off
    Y2 := TRUE;  // Green light on (free to go)
END_IF;
